# Yaml files 

## Ymal file for merge check 


```
name: Merge check

on:
  push:
    branches: [ main ]

jobs:
  merge-check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout main branch
        uses: actions/checkout@v2
        with:
          ref: main

      - name: Merge issue branch into main branch
        run: |
          git merge --no-commit --no-ff ${{ github.event.ref }}

      - name: Check for merge conflicts
        run: |
          if [[ -n $(git diff --name-only --diff-filter=U) ]]; then
            echo "There is a merge conflict, please resolve it before merging"
            git merge --abort
            exit 1
          else
            echo "No merge conflicts, merging changes into main"
            git commit -m "Merge ${{ github.event.ref }}" && git push origin main
          fi

```

This workflow uses the push trigger to run the job when a push event occurs on the main branch. It then uses the checkout action to fetch the latest changes from the main branch.

Next, the job uses the git merge command to merge the changes from the branch being pushed into the main branch, but with the --no-commit and --no-ff options to prevent a commit and force a merge commit. This allows the job to check for merge conflicts without actually committing any changes.

Then, the job uses the git diff command to check for any files with merge conflicts. If there are merge conflicts, the job prints an error message and aborts the merge using the git merge --abort command. If there are no merge conflicts, the job prints a success message, commits the changes using the git commit command with a commit message that includes the name of the branch being merged, and pushes the changes to the main branch using the git push command.

Note that this workflow only merges the changes into the main branch if there are no merge conflicts. If there are merge conflicts, the job stops and prints an error message. You will need to resolve the merge conflicts manually before running the workflow again.

