# Welcome to Git

##  Summary

What is Git and why do I care?

## Setting up Git

When setting up Git for the first time you need to configure your name and email address.  This is used to add your signature to commits to a repository.

``` bash
# Set your username:
git config --global user.name "FirstName LastName"
# Set your email address:
git config --global user.email "name@example.com"
```

## Basics

### Clone a Repository

``` bash
git clone https://github.com/PacketAnglers/avd-workshop.git
```

### Git Repository Status

``` bash
git status
```
If nothing has been modified, then you will see the following output.

``` text
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

If you modify a the contents of existing files or add/delete files from the repository, you will see something like:

``` text
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        newfile.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

### Stage File Changes

When you wish to update the repository files, you will need to stage the files. The above output gives you a clue as to the command needed to stage the changes.  You can specify individual files or specificy all files with a period `.`

To stage all file changes:

``` bash
git add .
```

Then check the status again to see what is staged and ready to be commited.

``` bash
git status
```

Output:

``` text
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   README.md
        new file:   newfile.txt
```

Now we have 2 files staged and ready to be commited to the `main` branch.

### Commit your changes

Now you can commit your staged changes with a comment:

???+ note

    Use a comment that will reflect the changes made so you can reference back to this commit in the future.

``` bash
git commit -m "Initial Commit Message"
```

Output:

``` text
[main 71765fe] Initial commit
 2 files changed, 2 insertions(+)
 create mode 100644 newfile.txt
```

## Intermediate

## Git / GitHub / GitLab

