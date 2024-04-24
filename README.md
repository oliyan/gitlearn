# Installation
1. Install GIT to your desktop.
2. Navigate to a folder, right click and select Open Git Bash here
   ![alt text](image.png)

## Settings
Let's talk about `git config` settings. 
### System Level 
Settings are applied for all users

### Global Level
Settings are applied for all the Repos of current user
```bash
git config --global user.name 'Ravisankar Pandian' #To add a user name for the git application.

git config --global user.email ravi@ravi.com #To add email for the git application 
```

### Local Level
Settings are applied only for the current Repository
# Usage

```bash
git init # to initialize a repository

# Then add files/folders to the workspace to edit them

git add <filename> # to add the modified/new files to the staging area
git add . # to add all the files to the staging area
git status # to show the status of our push/commit/staged changes
git commit # to commit the staged files for change
git commit -m 'commit message' # to commit along with a short commit message
git push # to push the committed files to the master branch?
git config --global user.name 'Ravisankar Pandian' # to add name for the github?
git config --global user.email 'ravi@ravi.com' # to add email for the github?

```
## Cheat Sheet
Click [here](git.pdf) to view the cheat sheet for git commands.