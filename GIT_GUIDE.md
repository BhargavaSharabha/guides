# The Complete Git & GitHub Guide
*From Beginner to Advanced - Master Version Control for Personal and Team Projects by Bhargava Sharabha Pagidimarri*

---

## Table of Contents
1. [Introduction](#introduction)
2. [Git Setup & Configuration](#git-setup--configuration)
3. [Core Git Commands](#core-git-commands)
4. [Advanced Git Features](#advanced-git-features)
5. [Collaboration with GitHub](#collaboration-with-github)
6. [Scenarios & Troubleshooting](#scenarios--troubleshooting)
7. [Best Practices](#best-practices)
8. [Visual Aids](#visual-aids)
9. [Appendix](#appendix)

---

## Introduction

### What is Git?
Git is a **distributed version control system** that tracks changes in your code over time. Think of it as a time machine for your projects that allows you to:
- Track every change made to your code
- Collaborate with others without overwriting each other's work
- Roll back to previous versions when things go wrong
- Work on multiple features simultaneously

### Why Version Control?
Before Git, developers would:
- Create backup folders like `project_backup_2023_01_15`
- Email code changes to teammates
- Manually merge different versions
- Lose track of what changed and when

Git solves these problems by providing a **single source of truth** for your project's history.

### GitHub's Role
GitHub is a web-based platform that hosts Git repositories and adds collaboration features:
- **Pull Requests**: Review and discuss code changes before merging
- **Issues**: Track bugs, feature requests, and project tasks
- **Actions**: Automate testing, building, and deployment
- **Projects**: Organize work with Kanban boards and automation

---

## Git Setup & Configuration

### Installing Git

#### Windows
```bash
# Download from https://git-scm.com/download/win
# Or use Chocolatey:
choco install git

# Or use Winget:
winget install --id Git.Git -e --source winget
```

#### macOS
```bash
# Using Homebrew:
brew install git

# Or download from https://git-scm.com/download/mac
```

#### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install git
```

#### Linux (CentOS/RHEL/Fedora)
```bash
# CentOS/RHEL:
sudo yum install git

# Fedora:
sudo dnf install git
```

### Initial Configuration

#### Set Your Identity
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

#### Verify Configuration
```bash
git config --list
# Shows all your Git configuration
```

#### Set Default Editor
```bash
# For VS Code:
git config --global core.editor "code --wait"

# For Vim:
git config --global core.editor "vim"

# For Nano:
git config --global core.editor "nano"
```

### SSH Key Setup for GitHub

#### Generate SSH Key
```bash
ssh-keygen -t ed25519 -C "your.email@example.com"
# Press Enter to accept default file location
# Enter a passphrase (recommended) or press Enter for no passphrase
```

#### Add SSH Key to SSH Agent
```bash
# Start SSH agent
eval "$(ssh-agent -s)"

# Add your SSH key
ssh-add ~/.ssh/id_ed25519
```

#### Add SSH Key to GitHub
```bash
# Copy your public key
cat ~/.ssh/id_ed25519.pub
# Copy the output and paste it in GitHub Settings > SSH and GPG keys
```

#### Test SSH Connection
```bash
ssh -T git@github.com
# You should see: "Hi username! You've successfully authenticated..."
```

---

## Core Git Commands

### Repository Basics

#### Initialize a New Repository
```bash
# Create a new directory and initialize Git
mkdir my-project
cd my-project
git init

# Or initialize in existing directory
cd existing-project
git init
```

#### Clone an Existing Repository
```bash
# Clone from HTTPS
git clone https://github.com/username/repository.git

# Clone from SSH
git clone git@github.com:username/repository.git

# Clone specific branch
git clone -b develop https://github.com/username/repository.git

# Clone to specific folder
git clone https://github.com/username/repository.git my-folder
```

#### Check Repository Status
```bash
git status
# Shows working directory status, staged changes, and untracked files

git status --short
# Compact status output
```

#### Stage Changes
```bash
# Stage specific files
git add filename.txt
git add folder/

# Stage all changes
git add .

# Stage specific types of files
git add *.js
git add src/

# Interactive staging
git add -i
```

#### Commit Changes
```bash
# Commit with message
git commit -m "Add user authentication feature"

# Commit with detailed message (opens editor)
git commit

# Stage and commit in one command
git commit -am "Fix login bug"
```

#### View History
```bash
# View commit history
git log

# Compact view
git log --oneline

# Graph view
git log --graph --oneline --all

# View specific file history
git log --follow filename.txt

# View changes in commits
git log -p
```

### Working with Files

#### Track New Files
```bash
# Create a new file
echo "# My Project" > README.md

# Add to Git
git add README.md
git commit -m "Add README file"
```

#### Ignore Files
Create a `.gitignore` file:
```bash
# Dependencies
node_modules/
vendor/

# Build outputs
dist/
build/
*.exe

# Environment files
.env
.env.local

# IDE files
.vscode/
.idea/
*.swp

# OS files
.DS_Store
Thumbs.db
```

#### Remove Files
```bash
# Remove file from Git tracking (keeps local file)
git rm --cached filename.txt

# Remove file completely
git rm filename.txt

# Remove directory
git rm -r folder/
```

### Branching & Merging

#### Create and Switch Branches
```bash
# Create new branch
git branch feature-name

# Create and switch to new branch
git checkout -b feature-name

# Modern way (Git 2.23+)
git switch -c feature-name

# Switch to existing branch
git checkout branch-name
git switch branch-name
```

#### List and Manage Branches
```bash
# List local branches
git branch

# List all branches (local and remote)
git branch -a

# List remote branches
git branch -r

# Delete local branch
git branch -d branch-name

# Force delete branch
git branch -D branch-name
```

#### Merge Branches
```bash
# Switch to target branch (usually main)
git checkout main

# Merge feature branch
git merge feature-name

# Merge with no-fast-forward (creates merge commit)
git merge --no-ff feature-name
```

#### Resolve Merge Conflicts

When conflicts occur, Git marks them in files:
```bash
<<<<<<< HEAD
// Your changes
=======
// Incoming changes
>>>>>>> feature-branch
```

Resolve manually, then:
```bash
# Add resolved files
git add filename.txt

# Complete merge
git commit
```

#### Rebase (Alternative to Merge)
```bash
# Switch to feature branch
git checkout feature-branch

# Rebase onto main
git rebase main

# Resolve conflicts if they occur
# Then continue rebase
git rebase --continue

# Or abort rebase
git rebase --abort
```

### Undoing Changes

#### Reset Commits
```bash
# Soft reset (keeps changes staged)
git reset --soft HEAD~1

# Mixed reset (keeps changes unstaged)
git reset HEAD~1

# Hard reset (discards all changes) ⚠️ DANGEROUS
git reset --hard HEAD~1

# Reset to specific commit
git reset --hard abc1234
```

#### Revert Commits
```bash
# Create new commit that undoes previous commit
git revert HEAD

# Revert specific commit
git revert abc1234

# Revert without committing
git revert -n abc1234
```

#### Stash Changes
```bash
# Stash current changes
git stash

# Stash with message
git stash push -m "WIP: working on feature"

# List stashes
git stash list

# Apply most recent stash
git stash pop

# Apply specific stash
git stash apply stash@{1}

# Drop stash
git stash drop stash@{0}
```

#### Cherry-pick Commits
```bash
# Apply specific commit to current branch
git cherry-pick abc1234

# Cherry-pick without committing
git cherry-pick -n abc1234
```

---

## Advanced Git Features

### References and Reflog

#### Understanding Refs
```bash
# HEAD points to current branch/commit
cat .git/HEAD

# View all refs
git show-ref

# View reflog (history of HEAD movements)
git reflog

# View reflog for specific branch
git reflog show main
```

#### Using Reflog for Recovery
```bash
# Find lost commit
git reflog

# Reset to specific reflog entry
git reset --hard HEAD@{2}
```

### Interactive Rebase

#### Squash Commits
```bash
# Interactive rebase last 3 commits
git rebase -i HEAD~3

# In the editor, change 'pick' to 'squash' for commits to combine
# Save and close to complete rebase
```

#### Rewrite Commit Messages
```bash
# Interactive rebase last commit
git rebase -i HEAD~1

# Change 'pick' to 'reword' to edit commit message
```

### Submodules

#### Add Submodule
```bash
# Add external repository as submodule
git submodule add https://github.com/username/library.git lib/library

# Initialize submodules in cloned repository
git submodule init
git submodule update
```

#### Work with Submodules
```bash
# Update all submodules
git submodule update --remote

# Remove submodule
git submodule deinit lib/library
git rm lib/library
rm -rf .git/modules/lib/library
```

### Git Hooks

#### Pre-commit Hook Example
Create `.git/hooks/pre-commit`:
```bash
#!/bin/sh
# Run tests before commit
npm test

# Run linting
npm run lint

# Exit with error if any command fails
if [ $? -ne 0 ]; then
    echo "Pre-commit checks failed. Commit aborted."
    exit 1
fi
```

Make it executable:
```bash
chmod +x .git/hooks/pre-commit
```

### Git Attributes

#### Line Ending Configuration
Create `.gitattributes`:
```bash
# Set line endings for different file types
*.js text eol=lf
*.jsx text eol=lf
*.ts text eol=lf
*.tsx text eol=lf
*.json text eol=lf
*.md text eol=lf
*.txt text eol=lf

# Binary files
*.png binary
*.jpg binary
*.gif binary
*.ico binary
*.pdf binary
```

---

## Collaboration with GitHub

### Repository Workflows

#### Forking Workflow
```bash
# 1. Fork repository on GitHub
# 2. Clone your fork
git clone https://github.com/your-username/repository.git

# 3. Add upstream remote
git remote add upstream https://github.com/original-owner/repository.git

# 4. Verify remotes
git remote -v
```

#### Keeping Fork Updated
```bash
# Fetch upstream changes
git fetch upstream

# Switch to main branch
git checkout main

# Merge upstream changes
git merge upstream/main

# Push to your fork
git push origin main
```

### Pull Requests

#### Create Feature Branch
```bash
# Create and switch to feature branch
git checkout -b feature/awesome-feature

# Make changes and commit
git add .
git commit -m "Add awesome feature"

# Push to your fork
git push origin feature/awesome-feature
```

#### Create Pull Request
1. Go to your fork on GitHub
2. Click "New Pull Request"
3. Select base branch (usually `main`)
4. Write descriptive title and description
5. Submit PR

#### PR Description Template
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
```

### Code Review Process

#### Review Checklist
- [ ] Code is readable and well-documented
- [ ] Tests are included and pass
- [ ] No debugging code or console.logs
- [ ] Error handling is appropriate
- [ ] Performance considerations addressed

#### Review Comments
```markdown
**Suggestion**: Consider extracting this logic into a separate function for reusability.

**Question**: Why do we need to check both conditions here?

**Bug**: This will throw an error if `user` is null.

**Nit**: Minor formatting issue - missing space after comma.
```

### Issue Management

#### Issue Templates
Create `.github/ISSUE_TEMPLATE/bug_report.md`:
```markdown
## Bug Description
Clear description of the bug

## Steps to Reproduce
1. Go to '...'
2. Click on '...'
3. See error

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- OS: [e.g. Windows 10]
- Browser: [e.g. Chrome 90]
- Version: [e.g. 1.0.0]

## Additional Context
Any other context about the problem
```

#### GitHub Projects
- Create Kanban boards for project management
- Automate issue movement based on PR status
- Use GitHub Actions for CI/CD integration

---

## Scenarios & Troubleshooting

### Common "Oops" Moments

#### I Committed to Wrong Branch
```bash
# 1. Stash or commit current changes
git stash

# 2. Switch to correct branch
git checkout correct-branch

# 3. Apply changes
git stash pop

# 4. Switch back and reset
git checkout wrong-branch
git reset --hard HEAD~1
```

#### I Pushed a Bad Commit
```bash
# 1. Revert the commit
git revert HEAD

# 2. Push the revert
git push origin main

# Alternative: Reset and force push (⚠️ DANGEROUS)
git reset --hard HEAD~1
git push --force origin main  # Only if you're sure!
```

#### I Lost a Commit
```bash
# 1. Find the commit in reflog
git reflog

# 2. Reset to the lost commit
git reset --hard HEAD@{5}  # Adjust number as needed
```

#### Detached HEAD State
```bash
# You're in detached HEAD state
git checkout main  # Return to main branch

# Or create new branch from current commit
git checkout -b new-branch
```

### Large File Management

#### Git LFS Setup
```bash
# Install Git LFS
git lfs install

# Track large file types
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "*.mp4"

# Add .gitattributes
git add .gitattributes
git commit -m "Configure Git LFS for large files"
```

#### Migrate Existing Files
```bash
# Convert existing files to LFS
git lfs migrate import --include="*.psd,*.zip,*.mp4"

# Force push to update remote
git push --force origin main
```

### Binary vs Text Files

#### Binary File Best Practices
- Use Git LFS for large binary files
- Keep binary files small when possible
- Consider if binary files are necessary in repo
- Use appropriate merge strategies

#### Text File Handling
```bash
# Configure merge strategy for specific files
git config merge.ours.driver true

# In .gitattributes
*.config merge=ours
```

### GitHub Actions Integration

#### Basic CI/CD Workflow
Create `.github/workflows/ci.yml`:
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Run linting
      run: npm run lint
    
    - name: Build project
      run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to production
      run: echo "Deploying to production..."
```

---

## Best Practices

### Commit Guidelines

#### Atomic Commits
```bash
# Good: Single logical change
git commit -m "Add user authentication with JWT"

# Bad: Multiple unrelated changes
git commit -m "Fix bugs and add features"
```

#### Commit Message Format
```bash
# Conventional Commits format
git commit -m "feat: add user authentication"
git commit -m "fix: resolve login validation bug"
git commit -m "docs: update API documentation"
git commit -m "style: format code according to style guide"
git commit -m "refactor: extract authentication logic"
git commit -m "test: add unit tests for auth service"
git commit -m "chore: update dependencies"
```

#### Branch Naming Conventions
```bash
# Feature branches
feature/user-authentication
feature/payment-integration

# Bug fixes
fix/login-validation
fix/memory-leak

# Hotfixes
hotfix/security-patch
hotfix/critical-bug

# Releases
release/v1.2.0
release/v2.0.0
```

### Security Best Practices

#### SSH Key Security
- Use strong passphrases
- Rotate keys regularly
- Use different keys for different services
- Store private keys securely

#### Repository Security
```bash
# Enable branch protection rules
# Require pull request reviews
# Require status checks to pass
# Restrict force pushes
# Require signed commits
```

#### GPG Signing
```bash
# Generate GPG key
gpg --full-generate-key

# Configure Git to use GPG
git config --global user.signingkey YOUR_GPG_KEY_ID

# Sign commits
git commit -S -m "Signed commit message"

# Verify signatures
git log --show-signature
```

### Code Review Etiquette

#### For Reviewers
- Be constructive and respectful
- Focus on code, not the person
- Explain the "why" behind suggestions
- Use questions to encourage learning

#### For Authors
- Respond to all review comments
- Don't take feedback personally
- Ask questions if feedback is unclear
- Thank reviewers for their time

---

## Visual Aids

### Branching Strategies

#### Git Flow
```
main     ●────────●────────●────────●
         │        │        │        │
develop  ●──●──●──●──●──●──●──●──●──●
         │  │  │  │  │  │  │  │  │  │
feature1 ●──●──●  │  │  │  │  │  │  │
         │  │  │  │  │  │  │  │  │  │
feature2    │  │  ●──●──●  │  │  │  │
         │  │  │  │  │  │  │  │  │  │
release      │  │  │  ●──●──●  │  │  │
         │  │  │  │  │  │  │  │  │  │
hotfix       │  │  │  │  │  ●──●──●  │
```

#### GitHub Flow
```
main     ●────────●────────●────────●
         │        │        │        │
feature1 ●──●──●──●        │        │
         │  │  │  │        │        │
feature2    │  │  ●──●──●──●        │
         │  │  │  │  │  │  │        │
feature3       │  │  │  │  ●──●──●──●
```

### Git Log Visualization

#### Basic Graph
```bash
git log --graph --oneline --all
```

Output:
```
* a1b2c3d (HEAD -> main) Merge pull request #123
|\
| * d4e5f6g (origin/feature-branch) Add new feature
|/
* b7c8d9e Update documentation
* e1f2g3h Initial commit
```

#### Advanced Visualization
```bash
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```

### Merge vs Rebase

#### Merge (Preserves History)
```
main     ●────────●────────●
         │        │        │
feature  ●──●──●──●        │
         │  │  │  │        │
         │  │  │  └──●─────┘
         │  │  │    │
         │  │  └────┘
         │  └────────┘
```

#### Rebase (Linear History)
```
main     ●────────●────────●
         │        │        │
feature  ●──●──●──●        │
         │  │  │  │        │
         │  │  │  └────────┘
         │  │  └────────────┘
         │  └────────────────┘
```

---

## Appendix

### Command Cheatsheet

#### Repository Management
| Command | Description |
|---------|-------------|
| `git init` | Initialize new repository |
| `git clone <url>` | Clone remote repository |
| `git remote add origin <url>` | Add remote origin |
| `git remote -v` | List remote repositories |

#### Basic Workflow
| Command | Description |
|---------|-------------|
| `git status` | Check working directory status |
| `git add <file>` | Stage changes |
| `git commit -m "message"` | Commit staged changes |
| `git push origin <branch>` | Push to remote |
| `git pull origin <branch>` | Pull from remote |

#### Branching
| Command | Description |
|---------|-------------|
| `git branch` | List local branches |
| `git branch <name>` | Create new branch |
| `git checkout <branch>` | Switch to branch |
| `git checkout -b <name>` | Create and switch to branch |
| `git merge <branch>` | Merge branch into current |

#### History & Debugging
| Command | Description |
|---------|-------------|
| `git log` | View commit history |
| `git log --oneline` | Compact history view |
| `git log --graph` | Graph view of history |
| `git reflog` | Reference history |
| `git show <commit>` | Show commit details |

#### Undoing Changes
| Command | Description |
|---------|-------------|
| `git reset --soft HEAD~1` | Undo last commit, keep changes |
| `git reset --hard HEAD~1` | Undo last commit, discard changes |
| `git revert <commit>` | Create new commit that undoes changes |
| `git stash` | Temporarily save changes |
| `git checkout -- <file>` | Discard file changes |

### Glossary

- **HEAD**: Pointer to the current commit/branch
- **Origin**: Default name for the remote repository you cloned from
- **Upstream**: The original repository in a fork workflow
- **Fast-forward merge**: When no new commits exist on the target branch
- **Conflict**: When Git can't automatically merge changes
- **Stash**: Temporary storage for uncommitted changes
- **Rebase**: Replaying commits on top of another branch
- **Cherry-pick**: Apply specific commits to current branch
- **Detached HEAD**: When HEAD points to a commit instead of a branch

### Useful Aliases

Add these to your `~/.gitconfig`:
```bash
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    ca = commit -a
    unstage = reset HEAD --
    last = log -1 HEAD
    visual = !gitk
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    ll = log --oneline --graph --all
```

### Recommended Tools

#### GUI Clients
- **GitKraken**: Cross-platform Git GUI with advanced features
- **SourceTree**: Free Git GUI by Atlassian
- **GitHub Desktop**: Simple Git client for beginners
- **VS Code Git**: Integrated Git support in VS Code

#### Command Line Tools
- **tig**: Text-mode interface for Git
- **lazygit**: Simple terminal UI for Git
- **git-flow**: Git extensions for Git Flow workflow

#### GitHub Extensions
- **GitHub CLI**: Command line interface for GitHub
- **GitHub Desktop**: Desktop application for GitHub
- **GitHub Actions**: CI/CD automation

### Official Resources

- [Git Documentation](https://git-scm.com/doc)
- [GitHub Guides](https://guides.github.com/)
- [GitHub Learning Lab](https://lab.github.com/)
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
- [GitHub CLI](https://cli.github.com/)

### Community & Support

- [Stack Overflow](https://stackoverflow.com/questions/tagged/git) - Tagged with 'git'
- [GitHub Community](https://github.com/orgs/community/discussions)
- [Reddit r/git](https://www.reddit.com/r/git/)
- [GitHub Issues](https://github.com/git/git/issues) - For Git itself

---

## Conclusion

Congratulations! You've completed the comprehensive Git and GitHub guide. You now have the knowledge and skills to:

✅ **Manage personal projects** with confidence  
✅ **Collaborate effectively** with teams  
✅ **Handle complex scenarios** and troubleshoot issues  
✅ **Follow best practices** for professional development  
✅ **Use advanced features** to optimize your workflow  

Remember: **Git is a tool that improves with practice**. Start with the basics, experiment with different workflows, and gradually incorporate advanced features into your daily routine.

**Happy coding and happy collaborating! 🚀**

---

*This guide is a living document. Git and GitHub evolve constantly, so stay updated with the latest features and best practices.*
