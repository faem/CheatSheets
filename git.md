# GIT CheatSheet

### Configuration

`git --version`

`git config --global user.name "name"`

`git config --global user.email "mail"`

`git config --list`


### .gitignore file

Create .gitignore file to ignore the file that we don't want in git, put the file name in the .gitignore file. These files will not be added in git.

### Clone a repo or branch

`git clone <ssh> <dir name where it will be cloned (optional)>`

`git clone -b <branch name> <ssh> <dir name where it will be cloned (optional)>`


### Prepare changes to commit

Stage all file: `git add . `

Stage all files: `git add *`

Stage all files (include deleted files too): `git add -A`

Stage all files with specified extension: `git add *.<file extension>`

### Commit

`git commit -m "commit message"`

### View History

shows status of tracked/untracked files: `git status`

shows the author info and commit made by him: `git log`

shows some abbreviated stats for each commit: `git log --stat`

shows specified no of entries: `git log -<number of entiries to show>`

shows what content has changed: `git log -p`


compelte workthrough of all command performed: `git reflog`

shows what is changed from the first of the commit: `git diff` 
git diff --cached/--staged

### Branching

`git branch <branch name>`

`git checkout <branch name>`

`git branch`

`git branch -a`

`git branch --merged`

`git checkout <commit hash>`

origin will be the cloned repository in github: `git pull origin master`

push changes to remote branch (origin): `git push origin master`

stage all changed files and then commit. Combination of git add -a and git commit: `git commit -a -m "<commit message>"`

(for 1 person) `git commit --amend -m "new message"`

`git cherry-pick <hash>`

reset to a commit: `git reset --soft/hard <hash of commit>`

deletes all untracked file(f) and directory(d): `git clean -df` 


like amend but for more person. history remains intake: `git revert <hash of commit>`

diff between 2 commit: `git diff <hash1> <hash2>`

create a branch and checkout it (-b flag) `git checkout -b <branch name>` 
<<<<<<< HEAD

delete branch, must not be checked out. (-d flag): `git branch -d <branch name>`

toggle with previous branch: `git checkout - ` 

### Reset to a branch

puts all file from being staged to untracked: `git reset`

`git reset --<mode> HEAD~<number>`

Here mode is:

=======

delete branch, must not be checked out. (-d flag): `git branch -d <branch name>`

toggle with previous branch: `git checkout - ` 

### Reset to a branch

puts all file from being staged to untracked: `git reset`

`git reset --<mode> HEAD~<number>`

Here mode is:

>>>>>>> tempBranch
hard (<number> of commits are deleted from HEAD)

soft (<number> of commits are STAGED from HEAD)

mixed (<number> of commits are UNSTAGED from HEAD). 

The default mode is mixed.

### Stashing

stash currently made changes: `git stash`

see all stash: `git stash list`

take top stash, apply and delete: `git stash pop` 

apply specific stash: `git stash apply <stash name>`

### Merge and Rebase

`git rebase -i`

`git merge <branch name>`

### Remove files from being Staged
`rm <file name>`

`git rm <file name>`

### Add changes to last commit

`git commit --amend --no-edit`
`git push -f origin some_branch` 


### Renaming files

`git mv file_from file_to`


### Delete branch

Remote:

`git push <remote_name> --delete <branch_name>`

Local:

`git branch -d branch_name`

### revert a specific file

`git checkout -- <filename>`

### Tags

show all tags: `git tag`

annotated tag: `git tag -a <tag name> -m "<tag message>"`

lightweight tag: `git tag <tag name>`

show info about the tag: `git show <tag name>`

tag previous commit: `git tag -a <tag name> <commit hash>`

push all tags to remote: `git push origin --tags`

push specific tag to remote: `git push origin <tag name>`

delete tag: `git tag -d <tag name>`

update deletion in remote: `git push origin :refs/tags/<tag name>`

to make change to a previous tag make a branch first: `git checkout -b <branch name> <tag name>`


### TODO

git mergetool

rebasing