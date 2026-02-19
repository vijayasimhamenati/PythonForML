# 🔄 Git Sync Guide: Local ↔ Google Colab ↔ GitHub

This guide outlines the "Google Drive Mirror" approach to maintain a seamless synchronization between your Local PC, Google Colab, and a remote GitHub repository. 



## 🏗️ 1. The Architecture (The "Sync Triangle")
* **Local PC:** Files live on your hard drive. You use standard Git commands.
* **Google Drive:** Acts as a persistent cloud hard drive for your repository.
* **Google Colab:** Acts as an ephemeral, high-powered CPU/GPU that mounts your Google Drive to edit the files. 
* **GitHub:** The central hub connecting the Local PC and Google Drive.

---

## 🔑 2. Prerequisites & Authentication

To allow Colab to push to GitHub without prompting for passwords:

1. **Generate a Personal Access Token (PAT):**
   * Go to GitHub > Settings > Developer Settings > Personal Access Tokens (Classic).
   * Generate a new token with `repo` scope.
2. **Store Secrets in Colab:**
   * Open Colab, click the **🔑 Secrets** icon on the left sidebar.
   * Add the following secrets and toggle **"Notebook access" to ON**:
     * `GITHUB_TOKEN`: Your generated PAT (`ghp_...`).
     * `GITHUB_USERNAME`: Your GitHub username.
     * `GITHUB_EMAIL`: Your GitHub email address.

---

## 🚀 3. Initial Setup (Day 1 Only)

Run this snippet in a temporary Colab cell to establish the repository in your Google Drive. *You only need to do this once per repository.*

```python
import os
from google.colab import drive
from google.colab import userdata

# 1. Mount Drive
drive.mount('/content/drive')

# 2. Retrieve Secrets
token = userdata.get('GITHUB_TOKEN')
username = userdata.get('GITHUB_USERNAME')

# 3. Define Path & Clone
# CHANGE 'Colab_Projects' to your preferred Drive folder
drive_path = '/content/drive/MyDrive/Colab_Projects'
os.makedirs(drive_path, exist_ok=True)
os.chdir(drive_path)

repo_name = "your-repo-name" # CHANGE THIS
clone_url = f"https://{token}@[github.com/](https://github.com/){username}/{repo_name}.git"

!git clone {clone_url}
print(f"✅ Cloned {repo_name} into {drive_path}")

```

---

## 🔄 4. The Daily Workflow (Day 2 & Beyond)

### Step 4A: The "Smart Header" Script

Place this script in the **very first cell** of your Jupyter Notebook. It automatically detects if you are in Colab, mounts Drive, navigates to your repo, and secures your Git identity locally (preventing the "Author identity unknown" error).

```python
import os
import sys

# --- CONFIGURATION ---
REPO_PATH = '/content/drive/MyDrive/Colab_Projects/your-repo-name' # CHANGE THIS

def setup_colab_env():
    if 'google.colab' in sys.modules:
        print("🟢 Colab Environment Detected")
        from google.colab import drive
        from google.colab import userdata
        
        # Mount Drive
        if not os.path.exists('/content/drive'):
            drive.mount('/content/drive')
            
        # Navigate to Repo
        try:
            os.chdir(REPO_PATH)
            print(f"📂 Working Directory: {os.getcwd()}")
            
            # Set Git Identity Locally (Persists in Drive's .git folder)
            email = userdata.get('GITHUB_EMAIL')
            name = userdata.get('GITHUB_USERNAME')
            os.system(f'git config user.email "{email}"')
            os.system(f'git config user.name "{name}"')
            print("👤 Git identity secured.")
            
        except FileNotFoundError:
            print(f"🔴 ERROR: Path not found. Did you run the Day 1 clone setup?")
    else:
        print("🔵 Local Environment Detected. Skipping Drive mount.")

setup_colab_env()

```

### Step 4B: Terminal Operations

Once the Smart Header is executed, open the **Colab Terminal** (`>_` icon) and follow standard Git practices:

1. **Start of Session:** Pull changes made locally.
```bash
git pull

```


2. **During Session:** Work normally. Press `Ctrl+S` frequently to save the `.ipynb` file to Google Drive.
3. **End of Session:** Stage, commit, and push back to GitHub.
```bash
git add .
git commit -m "feat: updated LangChain models and logic"
git push

```



---

## ⚙️ 5. Advanced Configurations

### Changing the Origin URL dynamically

If your token expires or you rename the repository, update the remote URL using your Colab secrets. Run this in a notebook cell:

```python
import os
from google.colab import userdata

token = userdata.get('GITHUB_TOKEN')
username = userdata.get('GITHUB_USERNAME')
repo_name = "your-repo-name" # CHANGE THIS

new_url = f"https://{token}@[github.com/](https://github.com/){username}/{repo_name}.git"
os.system(f'git remote set-url origin {new_url}')
print("✅ Origin URL updated securely.")

```

### Verifying Git Configuration

To ensure your Git identity is saving to the Drive (`.git/config`) and not the temporary Colab VM (`--global`), run:

```bash
git config --list --show-origin

```

*Look for `file:.git/config` next to your username/email.*
