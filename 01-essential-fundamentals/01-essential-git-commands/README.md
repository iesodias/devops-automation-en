# Hands-on Lab: Basic Git with GitHub

This lab will guide you through the basic Git commands and concepts with a focus on real GitHub usage.
We will practice:

- Installation
- Clone repository
- Add files
- Make commits
- Push to GitHub
- Create branch
- Merge
- Create Pull Request

---

## 1. Git Installation

### Windows:
- Download from: https://git-scm.com/download/win
- Install with default options

### Linux (Debian/Ubuntu):
```bash
sudo apt update
sudo apt install git -y
```

### macOS (using Homebrew):
```bash
brew install git
```

Check version:
```bash
git --version
```

Configure your name and email:
```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

---

## 2. Clone a repository

Create a repository on GitHub with a README.
Then, clone with:

```bash
git clone https://github.com/your-username/repo-name.git
cd repo-name
```

---

## 3. Add files

Create a file:
```bash
echo "# My project" > project.md
```

Add the file:
```bash
git add project.md
```

---

## 4. Make commit

```bash
git commit -m "Add initial project.md"
```

---

## 5. Push to GitHub

```bash
git push origin main
```

> Use "main" or "master" depending on your repository's main branch name.

---

## 6. Create branch

```bash
git checkout -b new-feature
```

Add something to the file and commit:
```bash
echo "Adding new functionality" >> project.md
git add project.md
git commit -m "New feature added"
```

---

## 7. Push the new branch

```bash
git push origin new-feature
```

---

## 8. Create Pull Request on GitHub

- Access the repository on GitHub
- Click "Compare & pull request"
- Add a title and description
- Click "Create pull request"

---

## 9. Merge on GitHub

- Access the "Pull requests" tab
- Click on the created PR
- Click "Merge pull request"
- Then "Confirm merge"

---

## 10. Update your local branch

Return to the main branch:
```bash
git checkout main
```

Update:
```bash
git pull origin main
```

---

# Intermediate Git Part

Now that you know the basic commands, let's explore more advanced features common in real projects:

## 11. Tag (versions)

Create a tag:
```bash
git tag v1.0.0
```

Push to GitHub:
```bash
git push origin v1.0.0
```

## 12. Ignore files (using .gitignore)

Create a file called `.gitignore`:
```txt
node_modules
*.log
.env
```

Add and commit normally:
```bash
git add .gitignore
git commit -m "Add .gitignore"
```

## 13. View modified files between commits

```bash
git diff <old-commit> <new-commit>
```

## 14. Configure an additional remote repository

```bash
git remote add upstream https://github.com/another-repo.git
```

Fetch changes:
```bash
git fetch upstream
```

Merge with your branch:
```bash
git merge upstream/main
```

---

## Automating Git with Custom Function on Linux (using `vi`)

### 1. Create the functions folder
```bash
mkdir -p ~/functions
```

---

### 2. Create the file with the function using `vi`
```bash
vi ~/functions/functions
```

Inside `vi`, follow these steps:

1. Press `i` to enter insert mode
2. Paste the function below:
   ```bash
    function d() {
      git add .
      git commit -m "$1"
      git push
    }
   ```
3. Press `ESC` to exit insert mode
4. Type `:wq` and press `ENTER` to save and exit

---

### 3. Edit `.bashrc` with `vi`
```bash
vi ~/.bashrc
```

1. Press `G` to go to the end of the file
2. Press `o` to add a new line
3. Type:
```bash
source ~/functions/functions
```
4. Press `ESC`
5. Type `:wq` and press `ENTER` to save and exit

---

### 4. Reload the terminal
```bash
source ~/.bashrc
```

---

### 5. Test the function
Inside a Git repository, run:
```bash
d
```

The function will run:
```bash
git add .
git commit -m "automatic commit"
git push
```