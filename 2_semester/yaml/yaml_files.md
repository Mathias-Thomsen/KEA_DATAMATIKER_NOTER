# Yaml files 

## Yaml file for merge check 


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


## Yaml file that check if there is any merge conflicts when you push the issue to the parent branch 

```
name: Merge Conflict Check

on:
  push:
    branches:
      - '**'

jobs:
  merge-conflict-check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout branch
        uses: actions/checkout@v2
      - name: Check for merge conflicts
        run: |
          git merge --no-commit --no-ff HEAD^
          if [ -n "$(git diff --name-only --diff-filter=U)" ]; then
            echo "Merge conflicts detected, please resolve before merging"
            exit 1
          fi
```
This workflow uses the push event to trigger the job on any branch. It uses the checkout action to fetch the branch you pushed to.

Next, the job uses the git merge command to attempt a merge between your branch and its parent branch, with the --no-commit and --no-ff options to prevent a commit and force a merge commit. This allows the job to check for merge conflicts without actually committing any changes.

The HEAD^ argument tells Git to merge the changes in your branch from the point where it diverged from its parent branch.

Finally, the job uses the git diff command to check for any files with merge conflicts. If there are merge conflicts, the job prints an error message and exits with a non-zero status code, which prevents the branch from being merged.

Note that this workflow only checks for merge conflicts and does not merge any changes. If there are merge conflicts, you will need to resolve them manually before merging your changes into the branch.


## This ymal file triggers when you push to main and it check if the code can build with maven. 

```
name: Build and push test

on: [push]

jobs:
 
  maven:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Output state
        run: |
          pwd
          ls
          env | sort
      - name: Set up Java
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'adopt'
      - name: Build with Maven
        run: mvn --batch-mode --update-snapshots package
      - name: Run the Maven verify phase
        run: mvn --batch-mode --update-snapshots verify  
```
This YAML file defines a GitHub Actions workflow named Build and push test that runs a Maven build and verifies the build using the mvn command. The workflow is triggered on every push to any branch in the repository.

Here's a breakdown of the workflow:

    The on key specifies that the workflow should be triggered on the push event.
    The jobs key defines a single job named maven that runs on an ubuntu-latest virtual machine.
    The uses key in the first step downloads the latest version of the repository using the actions/checkout action.
    The run key in the second step outputs the current working directory, a list of the files in the directory, and the environment variables that are available to the job.
    The uses key in the third step sets up the Java environment using the actions/setup-java action.
    The run key in the fourth step runs the Maven build in batch mode with the --update-snapshots option and the package goal to compile and package the project.
    The run key in the fifth step runs the Maven verify phase in batch mode with the --update-snapshots option to verify the build.

Overall, this workflow sets up a basic Maven build and test environment using GitHub Actions. You can customize it further to suit your specific project requirements.

