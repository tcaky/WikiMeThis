# Miscellaneous GH commands
This is so I have a quick reference to commands as I am learning (still don't have many of them memorized)

## Setting up user.name and user.email
See https://stackoverflow.com/a/46986131/1601161
```bash
#  global username/email configuration
git config --global user.name "FIRST_NAME LAST_NAME"
git config --global user.email "user@email.com"

# Can also set them local to the repo
# this will add them to the repo/.git/config file
git config user.name "FIRST_NAME LAST_NAME"
git config user.email "user@email.com"

# Show configuration
git config --list
```
# List releases from CLI
```bash
gh release list
```
# Add a file to an existing release
```bash
gh release upload RELEASE_TAG PATH_TO_FILE
```

## Show current branch
```bash
git branch --show-current
# or
git status
```

## Merge changes from main into current working branch
```bash
git pull origin main
```
OR

## Rebase from main (not sure if better to merge or rebase)
```bash 
# ensure on the correct brand
git checkout your-branch
# or check using status and then you may not need to checkout if you are already on the branch

# now rebase....
# This will apply the changes from main on top of your 
# branch, creating a cleaner history compared to a merge.
git fetch origin
git rebase origin/main
```