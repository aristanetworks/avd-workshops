# Welcome to Git

![Git Logo Dark](assets/images/git_logo_darkbg.png#only-dark){: style="width:200px"}
![Git Logo Light](assets/images/git_logo_lightbg.png#only-light){: style="width:200px"}

In this section we will explore a brief introduction of Git. We will cover the installation and explore basic commands used with Git.

## Topics

- Introduction to Git
  - What is Git and why should I use it?
- Installation & Setup
- The Basics of Git
  - My First Git Commands - Init and Status
  - Stage Changes
  - Commit Changes
- Intermediate Concepts
  - Fork a Repository
  - Clone a Repository
  - Creating a Branch
  - Push/Pull Changes
  - Pull Request

## Introduction to Git

Git is the most commonly used version control system. Git tracks changes to files, so you have a record of what has changed, and you can revert to specific versions. Changes to files are tracked by storing snapshots of the files over time. In the example below we have files A, B, and C. Over time the content of those files change. You are able to roll back to any previous commit or snapshot.

![Git Snapshots](assets/images/git_snapshots.png){: style="width:800px"}

Git makes collaboration easy, allowing multiple people to merge their changes into one source. Regardless of whether you work solo or as part of a team, Git will be useful for you.

## Installation & Setup

### Installation

Git has been pre-installed in your ATD Lab environment. If you need to install Git on another system, instructions are provided at the links below.

Download Git - [https://git-scm.com/downloads](https://git-scm.com/downloads)

Configuration - [https://git-scm.com/book/en/v2/Getting-Started-Installing-Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

### Setup

When setting up Git for the first time you need to configure your Identity with a name and email address. This is used to add your signature to commits. Also set the default branch name to `main`.

From the Terminal in your ATD Lab Progammability IDE running the following commands.

``` bash
# Set your username:
git config --global user.name "FirstName LastName"

# Set your email address:
git config --global user.email "name@example.com"

# Set default branch name to `main`
git config --global init.defaultBranch main
```

#### Progammability IDE

![Programmability IDE](assets/images/git_programmability_ide.png){: style="width:1000px"}

Verify your configuration:

``` bash
git config --global --list
```

## The Basics of Git

### My First Git Commands - Init and Status

#### Initialize Repository

The first thing you need to do to create a version controlled repository is to create a directory and initialize it. Initialize the following directory: `/home/coder/project/labfiles/project1/`.

``` bash
# From a Linux terminal
mkdir /home/coder/project/labfiles/project1/

# Change to the new directory
cd /home/coder/project/labfiles/project1/

# Initialize
git init
```

The directory is now initialized as a Git repository and the following hidden directory `/home/coder/project/labfiles/project1/.git/` was created. This is a special version control directory of the files in your repository.

#### Git Repository Status

``` bash
git status
```

If nothing has been modified, then you will see the following output.

``` text
On branch main

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

Once you add, delete, or modify any files, you will see similar output below. Here we are creating an empty file `newfile.txt` with the `touch` command.

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

### Stage Changes

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

### Commit Stages

Now you can commit your staged changes with a comment:

???+ note

    Use a comment that will reflect the changes made so you can reference back to this commit in the future.

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

## Intermediate Level

Before proceeding further, make sure you are logged into your active GitHub account.

If you do not have a GitHub account, you can create one **[here](https://github.com/join)**.

### Create a GitHub Personal Access Token

To push or copy your local repository to GitHUb you will need a Personal Access Token. From your **GitHub** account, click through the following path to generate a new personal access token.  **Profile --> Settings --> Developer Settings --> Personal Access Tokens --> Tokens (classic) --> Generate new token (classic)**

- Give the token a meaningful name by setting Note: `MyNewToken`
- Set the Expiration: 30 days (default)

Select the scopes you'd like to grant this token. To use your token to access repositories from the command line, select `repo`. A token with no assigned scopes can only access public information.

The click `Generate token` at the bottom of the page.  **Copy and save the token in a secure place. YOU WILL NOT BE ABLE TO SEE THE TOKEN AGAIN**.

![Git Token Scopes](assets/images/git_token_scopes.png){: style="width:500px"}

Detailed instructions can be found [here](https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).

### Fork a Repository

A fork is a copy of a repository that you manage. Forks let you make changes to a project without affecting the original repository. You can fetch updates from or submit changes to the original repository with a pull request.

We have created an example repository for this workshop that you can fork to make your own copy. We will make changes to this copy of the original repository.

#### Steps to Fork the example repository

1. On GitHub.com, navigate to the [Packet Anglers workshop](https://github.com/PacketAnglers/workshops) repository.
2. In the top-right corner of the page, click Fork.
![Fork](assets/images/git_fork.png){: style="width:800px"}
3. Select an owner for the forked repository.
4. Set repository name. By default, forks are named the same as their upstream repository.
5. Optionally, add a description of your fork.
6. Click the Fork button at the bottom

Next up... Clone this forked repository to your local host machine.

### Clone Repository

Cloning a repository allows us to make a local copy of a project that resides on GitHub. In the previous step, you forked a repo to your local GitHub account. Navigate to the forked repository in GitHub. From there, click on the green code button to get the URL of the forked repository.

![Git Clone](assets/images/git_clone.png){: style="width:400px"}

Clone this repository into the current directory of your local machine:

``` bash
# replace with the URL from your forked repo
git clone https://github.com/xxxxxxx/avd-workshop.git
```

Now change into the new cloned directory.

``` bash
cd avd-workshop
```

Verify where the remote copy of this clones lives.

``` bash
git remote -v
```

### Creating a Branch

A branch in Git is pointer to one of your commits. The default branch name in Git is `main`. As you make commits, the pointer moves to the latest commit on the current branch. Every time you commit, the branch pointer moves forward automatically.

Git branches are used to develop new features, without affecting the `main` branch. For example, if I wanted to play around with a new feature for my project, I might create a new branch called, `new-feature`.

![Branch](assets/images/git_branch.png)

You can see the current branch by typing:

``` bash
git branch
```

To create a new branch from the current branch pointer, type:

``` bash
git branch new-feature
```

To checkout or switch to this new branch, type:

``` bash
git checkout new-feature
```

Now you can modify existing files and commit the changes to this new branch. This tees up for the next section, `Pull Request`.

### Push/Pull Changes to GitHub

Once you modify the files in your new branch, you can stage & commit the changes and push them to the remote fork on GitHub.

``` bash
# stage all the changes
git add .

# commit them
git commit -m "updated content with new feature"

# push to remote repo on GitHub
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

![Git Commit](assets/images/git_commit.png){: style="width:800px"}

### Pull Request

A Pull Request in Git allows a contributor (you) to ask a maintainer (owner) of origin repository to review code changes they want to merge into a project. Once a pull request is opened, you can discuss and review the potential changes with collaborators and add follow-up commits before your changes are merged into the base branch.

Once all changes have been agreed upon, the maintainer of the original repo will merge your changes into the `main` branch. At this point, your code changes are live and visible in the origin project repo.

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
