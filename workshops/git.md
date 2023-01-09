![Git Logo](assets/images/git_logo.png)

# Git - Section Content

In this section we will cover and introduction and installation of Git and explore basic commands to used with Git. Intermediate Concepts are also covered to further your knowledge of Git.

## Topics

- Introduction to Git
    - What is Git and why should I use it?
- Installation & Setup
- The Basics of Git
  - My First Git Commands - Init and Status
  - Stage Changes
  - Commit Changes
- Intermediate Concepts
  - Clone a Repository
  - Push/Pull Changes
  - Fork a Repository
  - Creating a Branch
  - Pull Request

## Introduction to Git

###  What is Git and why should I use it?

Git is a DevOps tool used for source code management. It is a free and open-source version control system used to handle small to very large projects efficiently. Git is used to tracking changes in the source code, enabling multiple developers to work together on non-linear development. Git can automatically merge the changes, so two people can even work on different parts of the same file and later merge those changes without loosing each other’s work!

## Installation & Setup

### Installation

Download Git - [https://git-scm.com/downloads](https://git-scm.com/downloads)

Configuration - [https://git-scm.com/book/en/v2/Getting-Started-Installing-Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

### Setup

When setting up Git for the first time you need to configure your Identity with a name and email address.  This is used to add your signature to commits.

``` bash
# Set your username:
git config --global user.name "FirstName LastName"
# Set your email address:
git config --global user.email "name@example.com"
```

Verify your configuration:

``` bash
git config --global --list
```

## The Basics of Git

### My First Git Commands - Init and Status

#### Initialize Repository

The first thing you need to do to create a version controlled repository is to create a directory and initialize it.  Initialize the following directory: `/home/user/git_projects/project1/`.

``` bash
# From a Linux terminal
mkdir /home/user/git_projects/project1/

git init
```

The directory is not initialized as a Git repository and the following hidden directory `/home/user/git_projects/project1/.git/` was created.  This is a special directory used version control of the files in your repository.

#### Git Repository Status

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

### Stage Changes

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

### Commit Stages

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

## Intermediate Level

### Clone a Repository

GitHub is a public site to store code repositories. Navigate to the following example repository [https://github.com/PacketAnglers/avd-workshop](https://github.com/PacketAnglers/avd-workshop). Click on the green `Code` button and copy the URL.

![Git Clone](assets/images/git_clone.png)

Clone this repository into the current directory of your local machine:

``` bash
git clone https://github.com/PacketAnglers/avd-workshop.git
```


### Push/Pull Changes to GitHub

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

### Fork a Repository

A fork is a copy of a repository that you manage. Forks let you make changes to a project without affecting the original repository. You can fetch updates from or submit changes to the original repository with a pull request.

On GitHub.com, navigate to the `PacketAnglers/avd-workshop` repository.

In the top-right corner of the page, click Fork.

![Fork](assets/images/git_fork.png)

Select an owner for the forked repository.

By default, forks are named the same as their upstream repositories. ...

Optionally, add a description of your fork.

### Creating a Branch

A branch in Git is pointer to one of your commits. The default branch name in Git is `main`. As you make commits, the pointer moves to the latest commit onthe current branch. Every time you commit, the branch pointer moves forward automatically.

Git branches are used to develop new features, without affecting the `main` branch. For example, if I wanted to play around with a new feature for my project, I might create a new branch called, `new-feature`.

![Branch](assets/images/git_branch.png)

### Pull Request