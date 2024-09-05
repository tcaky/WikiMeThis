# Miscellaneous GH commands
This is so I have a quick reference to commands as I am learning (still don't have many of them memorized)


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