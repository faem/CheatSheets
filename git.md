`git --version`

`git config --global user.name "name"`

`git config --global user.email "mail"`

`git config --list`


Create .gitignore file to ignore the file i don't want in git put the file name in the .gitignore file. This file will not be added in git

`git status`

`git add . `

`git reset <file-name>`

removes all file from being staged: `git reset`


`git commit -m "commit message"`

shows the author info and commit made by him: `git log`

`git log --stat`


shows what is changed from the first of the commit: `git diff` 

`git status `

`git add . `

`git add *`

`git add -A`

`git add *.<file extension>`

`git commit -m "message"`


origin will be the cloned repository in github: `git pull origin master`

push changes to remote branch (origin): `git push origin master`



`git branch <branch name>`

`git checkout <branch name>`

`git branch`

`git branch -a`

`git branch --merged`

`git branch -d <branch name>`

`git push origin --delete <branch name>`

`git merge <branch name>`

stage all changed files and then commit. Combination of git add -a and git commit: `git commit -a -m "<commit message>"`

(for 1 person) `git commit --amend -m "new message"`

`git cherry-pick <hash>`

reset to a commit: `git reset --soft/hard <hash of commit>`

deletes all untracked file(f) and directory(d): `git clean -df` 

compelte workthrough of all command performed: `git reflog`

like amend but for more person. history remains intake: `git revert <hash of commit>`

diff between 2 commit: `git diff <hash1> <hash2>`

`git clone <ssh> <dir name (optional)>`

`git clone -b <branch name> <ssh> <dir name>`

`git reset --hard HEAD`


create a branch and checkout it (-b flag) `git checkout -b <branch name>` 

delete branch, must not be checked out. (-d flag): `git branch -d <branch name>`

toggle with previous branch: `git checkout - ` 

take top stash, apply and delete: `git stash pop` 

`git rebase -i`