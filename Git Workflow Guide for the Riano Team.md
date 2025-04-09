This document outlines the standard Git commands and conventions for the Riano Team to manage repositories, branches, commits, and pull requests effectively. It aligns with the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) standard to ensure clear, consistent, and meaningful commit messages.

---
## 1. Remote Repository Management

### Adding a Remote Repository
- **Clone a repository**:  
  `git clone <url>`  
  Downloads a repository from the specified URL to your local machine.
- **Add a remote**:  
  `git remote add <short-name> <url>`  
  Links a remote repository with a shorthand name (e.g., `origin`).
- **View remotes**:  
  `git remote -v`  
  Lists all remotes with their URLs.
- **Rename a remote**:  
  `git remote rename <old-name> <new-name>`  
  Updates the shorthand name of a remote.

### Syncing with Remotes
- **Fetch changes**:  
  `git fetch <short-name> <branch>`  
  Retrieves updates from the remote without merging them.
- **Pull changes**:  
  `git pull <short-name> <branch>`  
  Fetches and merges updates from the remote into your local branch.
- **Fetch vs Pull**:  
  - `fetch`: Downloads updates but doesn’t apply them.  
  - `pull`: Downloads and applies updates.

### Pushing Changes
- **Push to remote**:  
  `git push <short-name> <branch>`  
  Uploads your local branch changes to the remote repository.

---

## 2. Committing Changes

### Basic Commits
- **Stage and commit all changes**:  
  `git commit -a -m "<message>"`  
  Automatically stages tracked files and commits them with a message.
- **Amend a commit**:  
  `git commit --amend -m "<new-message>"`  
  Updates the last commit with new changes or a revised message.

### Undoing Changes
- **Unstage a file**:  
  `git restore --staged <file-name>`  
  Removes a file from the staging area.
- **Discard changes in a file**:  
  `git restore <file-name>`  
  Reverts uncommitted changes in a specific file.

---

## 3. Stashing Changes

- **Stash uncommitted changes**:  
  `git stash`  
  Temporarily saves your changes without committing.
- **List stashes**:  
  `git stash list`  
  Shows all saved stashes.
- **Apply a stash with index**:  
  `git stash apply <index>`  
  Reapplies a specific stash (e.g., `git stash apply 0`).
- **Drop a stash**:  
  `git stash drop <index>`  
  Deletes a specific stash.
- **Clear all stashes**:  
  `git stash clear`  
  Removes all stashes.

---

## 4. Riano Team Workflow

### Step-by-Step Process
1. **Sync with Main/Develop**:  
   `git pull origin develop` or `git pull origin main`  
   Ensure your local branch is up-to-date with the main or develop branch.

2. **Create a Branch**:  
   `git branch <branch-name>`  
   - Example: `git branch deep-link`  
   - Naming: Use kebab-case (e.g., `deep-link`), avoid camelCase.  
   - Prefix examples:  
     - `feature-deep-link` (new features)  
     - `fix-deep-link` (bug fixes)

3. **Start a Feature (Optional Git Flow)**:  
   `git flow start feature deep-link`  
   Creates a feature branch following Git Flow conventions.

4. **Committing Changes**:  
   Use [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) for clarity:  
   `git commit -m "<type>(<scope>): <short-description> <task-code>"`  
   - **Types**:  
     - `feat`: New feature  
     - `fix`: Bug fix  
     - `core`: Core app changes  
   - **Scope**: Context (e.g., `login`, `imagePicker`, `app`).  
   - **Short Description**: Brief explanation of the change.  
   - **Task Code**: Task identifier (e.g., `1234`).  
   - Examples:  
     - `git commit -m "feat(imagePicker): add image picker 1234"`  
     - `git commit -m "fix(login): resolve auth issue 5678"`  
     - `git commit -m "core(app): update config 9012"`

5. **Push the Branch**:  
   `git push origin <branch-name>`  
   Uploads your branch to the remote repository.

6. **Create a Pull Request (PR)**:  
   - Submit your branch for review via your repository hosting platform (e.g., GitHub, GitLab).  
   - Reference the task code in the PR description.

7. **Delete the Branch**:  
   After the PR is merged, delete the branch:  
   - Local: `git branch -d <branch-name>`  
   - Remote: `git push origin --delete <branch-name>`

---

## 5. Commit Message Guidelines (Conventional Commits)

- **Structure**:  
  `<type>(<scope>): <short-description> <task-code>`  
- **Rules**:  
  - Keep descriptions concise and meaningful.  
  - Use lowercase for descriptions.  
  - Include the task code at the end if applicable.  
- **Examples**:  
  - `feat(login): add social login support 1234`  
  - `fix(imagePicker): fix crash on empty selection 5678`  
  - `core(app): optimize startup time 9012`

---

## 6. Best Practices
- **Pull frequently**: Stay in sync with `develop` or `main`.  
- **Small commits**: Commit often with focused changes.  
- **Descriptive branches**: Use prefixes like `feature-` or `fix-` for clarity.  
- **Review PRs**: Ensure code is reviewed before merging.

---

This guide ensures the Riano Team follows a consistent Git workflow while adhering to industry-standard conventions. For more details on Conventional Commits, visit [https://www.conventionalcommits.org/en/v1.0.0/](https://www.conventionalcommits.org/en/v1.0.0/).

--- 