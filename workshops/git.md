# Welcome to Git

<img src="assets/images/git_logo.png" alt="Git Logo" width="100"/>

##  What is Git and why should I use it?

Git is a DevOps tool used for source code management. It is a free and open-source version control system used to handle small to very large projects efficiently. Git is used to tracking changes in the source code, enabling multiple developers to work together on non-linear development. Git can automatically merge the changes, so two people can even work on different parts of the same file and later merge those changes without loosing each other’s work!

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

GitHub is a public site to store code repositories. Navigate to the following example repository [https://github.com/PacketAnglers/avd-workshop](https://github.com/PacketAnglers/avd-workshop). Click on the green `Code` button and copy the URL.

![Git Clone](assets/images/git_clone.png)

To clone this repository to your local machine:

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

Now these files are commited to your local copy of the repository.  To update the remote repository in GitHub you need to push the changes.

### Push Changes to GitHub

``` bash
git push
```

Output:

``` text
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 6 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 328 bytes | 328.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/PacketAnglers/avd-workshop.git
   07f3bd2..71765fe  main -> main
```

You should now see these changes and commit messages in GitHub.

![Git Commit](assets/images/git_commit.png)
## Intermediate

## Git / GitHub / GitLab
