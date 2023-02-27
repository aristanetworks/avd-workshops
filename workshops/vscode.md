# Welcome to VS Code

![VS Code Logo](assets/images/vscode_logo.png){: style="width:130px"}

## What is VS Code?

Visual Studio Code (VS Code), written by Microsoft, is a lightweight but powerful source code editor which runs on your desktop and is available for Windows, macOS and Linux. It comes with built-in support for JavaScript, TypeScript and Node.js and has a rich ecosystem of extensions for other languages and runtime environments (such as C++, C#, Java, Python, PHP, Go, .NET).

VS Code integrates directly with Git and allows Network Engineers to easily perform the following tasks:

- Edit Files
- Drag and Drop Files into the Explorer
- Visually Compare File Diffs
- Write & Test Scripts
- Open Terminals for various shells
- Execute various Git commands (such as: status, init, log, commit, switch, add, etc...)
- Connect to Remote Hosts

VS Code is your personal Git Genie making automation workflows easy. Once you learn to use VS Code, you will wonder why it took so long to get on the bandwagon.

## System Requirements

Visual Studio Code is a small download (< 200 MB) and has a disk footprint of < 500 MB. VS Code is lightweight and should easily run on today's hardware.

Recommend Hardware:

- 1.6 GHz or faster processor
- 1 GB of RAM

Support Platforms:

- OS X El Capitan (10.11+)
- Windows 8.0, 8.1 and 10, 11 (32-bit and 64-bit)
- Linux (Debian): Ubuntu Desktop 16.04, Debian 9
- Linux (Red Hat): Red Hat Enterprise Linux 7, CentOS 7, Fedora 34

## Download & Installation

VS Code downloads for Windows, Linux and Mac can be found [here](https://code.visualstudio.com/download).

## VS Code GUI Walk-through

The four main areas of the VS Code GUI are:

- Activity Bar
- Status Bar
- Editor
- Terminal Window

![VS Code Screenshot](assets/images/vscode_screenshot.png){: style="width:1000px"}

???+ note
    Your `Activity Bar` may have different icons for the extensions you have installed.

## Extensions

There are tons of extensions to enhance your experience with VS Code. Below is a list of recommended extensions that are commonly used. For a full list of available extensions visit the [VS Code Marketplace](https://marketplace.visualstudio.com/vscode).

- Markdown All-in-One (Yu Zhang)
- YAML (Red Hat)
- Better Jinja (Samuel Colvin)
- Peacock (John Papa)
- Remote - SSH (Microsoft)
- Dev Containers (Microsoft)

## Common things to know

### Open/Close Folders

To open a file or folder go to ```File > Open``` or use ```Cmd/Ctrl + o```. Once you have a folder open you can simply just click on file you wish to open from the Explorer in the Activity Bar. Note if you only single click the file when you go to open another file it will replace it in the editor, to keep a file open double click it. The difference can be noticed by looking at the editor tab title. The tab in italics will be replaced. As seen in the image below.

![VS Code Logo](assets/images/vscode_Editor_Open_Italics.png){: style="width:260px"}

To close a file you can click on the ```x``` of the editor tab, if you instead see a circle that means the changes to that file are unsaved.

To close the folder go to ```File > Close Folder``` or ```Cmd/Ctrl + k``` then hit ```f```.

### Save & Auto Save

By default VS Code will not auto save file changes. To configure auto save click ```File > Auto Save```. If you wish to save manually you can save by going to ```File > Save``` or ```Cmd/Ctrl + s```.

If a file has unsaved changes, the close button on the tab will be a circle/dot instead of the normal ```x```. As seen in the image below.

![VS Code Logo](assets/images/vscode_Unsaved_Changes.png){: style="width:130px"}

### Command Palette

The Command Palette lets you access several commands and settings. To open it go to ```View > Command Palette``` or hit ```Cmd/Ctrl + Shift + P```. There are a lot of powerful commands you can test. If you would like to read more go [here](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette). As an example you can run git commands like `git clone`.

### Start a Terminal

VS Code allows you to launch a terminal session from within the program. To access it you can either go up to ```Terminal > New Terminal``` or hit ```Cmd/Ctrl + Shift + ` ```. You can create multiple instances of terminals and they can be different types, i.e. bash, python, zsh(if you have it installed).

## Git Integration

In the previous section we ran all Git commands from the command-line. Now we will use VS Code to perform the same operations. First let's reset the `samplefiles` directory by removing the hidden sub-directory `.git`.  This will remove any version control settings for the repository.

``` bash
cd /home/coder/project/labfiles/samplefiles
rm -rf .git
```

## The Basics

### Initialize Repository

There are multiple ways to initialize a repository from VS Code. We will explore one of the methods.

First open the folder `/home/coder/project/labfiles/samplefiles` from within the VS Code Explorer.

![Folder Open](assets/images/vscode_open_folder.png){: style="width:400px"}

Re-open a new terminal window.

Next click on the `Source Control` icon in the Activity Bar, and then click `Initialize Repository`.

![Init Repo](assets/images/vscode_init_repo.png){: style="width:300px"}

This is equivalent to running `git init` from within that directory.

Several things just happened. VS Code is giving us a visual representation of the newly created Git repository.  The `Source Control` icon now shows a blue dot with a `6`.  This is indicates there a 6 untracked, represented by the capital `U` next to each file.

Also, in the bottom left corner of status bar we can see we are on the `main`* branch.

![status bar](assets/images/vscode_status_bar.png){: style="width:400px"}

## VS Code Workflow

Use VS Code to perform the following actions.

- Stage files
- Commit staged files
- Check the log - git log
- Verify current branches
- Create and switch to new branch called `change-usernames`
- Update username `arista` to `admin`
  - Use VS Code Replace in files
  - Show file diff
- Stage and commit changes to new branch
- Switch back to `main` branch
- Merge `change-usernames` into `main`
  - Command-Palette - git merge
- Verify files have new names in the `main` branch
- Be a good citizen and clean up old branch
- Publish Repo to GitHub

## Dev Containers

[Dev Container extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) allows you to use containers as a development environment. This is useful when several team members are working on a project together and wish to use the same development environment.

In the Command Palette, the option "Dev Containers: Create Dev Container..." should show up after the extension is installed. If a ".devcontainer" folder is already present in the root directory of the project, then you can choose the "Dev Containers: Reopen in Container" option. Dev Containers make managing a dev environnment much easier as you can include it into your git repository and when others go to make changes they will have the same environment, given that they use the dev container. No need to deal with different versions or missing software.

!!! Note
    Docker must be installed for Dev Containers to work.
