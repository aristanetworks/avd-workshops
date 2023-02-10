# Getting started with Git

![Git Logo Dark](assets/images/git_logo_darkbg.png#only-dark){: style="width:200px"}
![Git Logo Light](assets/images/git_logo_lightbg.png#only-light){: style="width:200px"}

## Introduction

In this section we will explore a brief introduction of Git. We will cover the installation and basic commands used with Git. Git is the most commonly used version control system. Git tracks changes to files allowing you to revert to specific versions. File changes are tracked by storing snapshots (commits) of the files over time. In the image below, the content of files A, B, and C change over time. Git allows you to roll back to any previous commit.

![Git Snapshots](assets/images/git_snapshots.png){: style="width:800px"}

Git makes collaboration easy by allowing multiple people to merge their changes into one source. Regardless of whether you work solo or as part of a team, Git will be useful for you.

## Installation & setup

### Installation

???+ note
    Git has been pre-installed in your ATD Lab environment. If you need to install and configure Git on another system, follow the instructions at the links above.

Download Git - [https://git-scm.com/downloads](https://git-scm.com/downloads)

Configuration - [https://git-scm.com/book/en/v2/Getting-Started-Installing-Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

### Setup

When setting up Git for the first time you need to configure your Identity with a name and email address. This is used to add your signature to commits. Additonally, set the default branch name to `main`.

From the Terminal in your ATD Lab Progammability IDE running the following commands.

``` bash
# Set your username:
git config --global user.name "FirstName LastName"

# Set your email address:
git config --global user.email "name@example.com"

# Set default branch name to `main`
git config --global init.defaultBranch main
```

#### Progammability IDE (VS Code)

![Programmability IDE](assets/images/git_programmability_ide.png){: style="width:1000px"}

Verify your configuration:

``` bash
git config --global --list
```

## Basic commands

### Git - init and status

#### Initialize directory as a Git repository

Create a directory called `/home/coder/project/labfiles/project1/` and initialize it as a repository (repo).

???+ note
    In the ATD Lab environment you only have permissions to create files and directories under `/home/coder/project/labfiles/`.

``` bash
# From a Linux terminal
mkdir /home/coder/project/labfiles/project1/

# Change to the new directory
cd /home/coder/project/labfiles/project1/

# Initialize
git init
```

The directory is now initialized as a Git repository and the following hidden sub-directory `/home/coder/project/labfiles/project1/.git/` was created. It holds version control information for your repository.

#### Git Repository Status

Check the current status of your repo.

``` bash
git status
```

Since this is a brand new repo you should see the following output.

``` text
On branch main

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

Once you add, delete, or modify any files, you will see similar output below. Here we are creating an empty file `newfile.txt` with the Linux `touch` command.  Now Git will show there are Untracked files.

``` text
➜  project1 git:(main) touch newfile.txt
➜  project1 git:(main) ✗ git status
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        newfile.txt

nothing added to commit but untracked files present (use "git add" to track)
```

### Stage your changes

When you are ready to commit your changes, you need to stage the files. The above output gives you a clue as to the command needed to stage the changes. You can specify individual files or add all files with a wildcard period `.`

To stage all file changes:

``` bash
git add .
```

Then check the status again to see what is staged and ready to be committed.

``` bash
git status
```

Output:

``` text
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   newfile.txt
```

Now we have a new file staged and ready to be committed to the `main` branch.

### Commit your changes

Now you can commit your staged changes with a comment:

???+ note
    Use a comment that will reflect the changes made so you can reference back to this commit in the future. Commit messages will show up in the log.

``` bash
git commit -m "First Commit"
```

Output:

``` text
[main (root-commit) 22e4d69] First Commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 newfile.txt
```

Now these files are committed to your local repository.

## Intermediate commands

Before proceeding further, make sure you are logged into your active GitHub account.

If you do not have a GitHub account, you can create one **[here](https://github.com/join)**.

???+ note
    In the ATD Lab, you will authenticate to GitHub using an 8 digit access code.  On other systems you will need a Personal Access Token. You may skip the next step if you are working in the ATD Lab IDE.

### Create a GitHub personal access token

To push your local repo to GitHUb you will need a Personal Access Token. From your **GitHub** account, click through the following path to generate a new personal access token.  **Profile --> Settings --> Developer Settings --> Personal Access Tokens --> Tokens (classic) --> Generate new token (classic)**

- Give the token a meaningful name by setting the **Note**: `MyNewToken`
- Set the **Expiration**: 30 days (default)

Select the scopes to grant to this token. To use your token to access repositories from the command line, select `repo`. A token with no assigned scopes can only access public information.

The click `Generate token` at the bottom of the page.  **Copy and save the token in a secure place. YOU WILL NOT BE ABLE TO SEE THE TOKEN AGAIN**.

![Git Token Scopes](assets/images/git_token_scopes.png){: style="width:500px"}

???+ note
    Detailed instructions for creating a Personal Access Token can be found [here](https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).

### Fork a repository

A fork is a copy of another repository that you can manage. Forks let you make changes to a project without affecting the original repository. You can fetch updates from or submit changes to the original repository with a pull request.

Fork the example [repository](https://github.com/PacketAnglers/workshops) to make your own copy. Then you can modify your copy of the original repository.

#### Steps to Fork the example repository

1. From GitHub.com, navigate to the [Packet Anglers workshop](https://github.com/PacketAnglers/workshops) repository.
2. In the top-right corner of the page, click Fork.
![Fork](assets/images/git_fork.png){: style="width:800px"}
3. Select an owner.
4. Set repository name. By default, forks are named the same as their upstream repository.
5. Optionally, add a description of your fork.
6. Click `Create fork` button at the bottom

Next up... Clone this forked repository to your local host machine.

### Clone forked repo to local host

Cloning a repository allows us to make a local copy of a project that resides on GitHub. In the previous step, you forked a repo to your local GitHub account. Navigate to the forked repository in GitHub. From there, click on the green code button to get the URL of the forked repository.

![Git Clone](assets/images/git_clone.png){: style="width:400px"}

Clone this repository into the current directory of your local machine. In the ATD Lab your current directory needs to be `/home/coder/project/labfiles/`.

``` bash
# replace with the URL from your forked repo
git clone https://github.com/xxxxxxx/workshops.git
```

Now change into the new cloned directory.

``` bash
cd workshops
```

Verify where the remote copy of this clones lives.

``` bash
git remote -v
```

### Creating a Branch

A branch in Git is pointer to one of your commits. The default branch name in Git is `main`. As you make commits, the pointer moves to the latest commit on the current branch. Every time you commit, the branch pointer moves forward automatically.

Git branches are used to develop new features, without affecting the `main` branch. For example, if you wanted to play around with a new feature for your project, you might create a new branch called, `new-feature`.

![Branch](assets/images/git_branch.png)

Verify current branch by typing:

``` bash
git branch
```

Create a new branch from the current branch pointer, type:

``` bash
git branch new-feature
```

Switch to this new branch, type:

``` bash
git switch new-feature
```

Now you can modify existing files and commit the changes to this new branch.

### Push/Pull Changes to GitHub

Once you modify the files in your new branch, you can stage & commit the changes and push them to the remote fork on GitHub.

``` bash
# stage all the changes
git add .

# commit them
git commit -m "updated content with new feature"

# push new branch to remote repo on GitHub
git push --set-upstream origin new-feature
```

Output:

``` text
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 16 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 348 bytes | 348.00 KiB/s, done.
Total 4 (delta 2), reused 1 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
remote:
remote: Create a pull request for 'new-feature' on GitHub by visiting:
remote:      https://github.com/mthiel117/workshops/pull/new/new-feature
remote:
To https://github.com/mthiel117/workshops.git
 * [new branch]      new-feature -> new-feature
Branch 'new-feature' set up to track remote branch 'new-feature' from 'origin'.
```

You should now see the new branch `new-feature` and commit messages in GitHub.

![Git Commit](assets/images/git_commit.png){: style="width:800px"}

The next step is to merge the `new-feature` branch into the `main` branch.  A Pull Request is used to do this.

### Pull request

A Pull Request in Git allows a contributor (you) to ask a maintainer (owner) of origin repository to review code changes you wish to merge into a project. Once a pull request is opened, you can discuss and review the potential changes with collaborators and add follow-up commits before your changes are merged into the `main` branch.

Once all changes have been agreed upon, the maintainer of the original repo will merge your changes. At this point, your code changes are visible in the origin project repo.

#### Steps to initiate a Pull Request

1. On GitHub.com, navigate to the main page of the repository.
2. In the "Branch" menu, choose the branch that contains your commits.
3. Above the list of files, click the Contribute drop-down and click Open pull request.
![PR](assets/images/git_pullrequest.png){: style="width:800px"}
4. Use the base branch dropdown menu to select the branch you'd like to merge your changes into, then use the compare branch drop-down menu to choose the topic branch you made your changes in.
![PR Info](assets/images/git_pullrequest_info.png){: style="width:800px"}
5. Type a title and description for your pull request.
6. To create a pull request that is ready for review, click Create Pull Request. To create a draft pull request, use the drop-down and select Create Draft Pull Request, then click Draft Pull Request.

This will generate a Pull Request on the main project repository. The owner/maintainer can then merge the pull request once all changes are satisfied.
