# Welcome to VS Code

![VS Code Logo](assets/images/vscode_logo.png){: style="width:130px"}

## What is VS Code?

Visual Studio Code (VS Code), written by Microsoft, is a lightweight but powerful source code editor which runs on your desktop and is available for Windows, macOS and Linux. It comes with built-in support for JavaScript, TypeScript and Node.js and has a rich ecosystem of extensions for other languages and runtime environments (such as C++, C#, Java, Python, PHP, Go, .NET).

VS Code integrates directly with Git and allows Network Engineers to easily perform the following tasks:

- Update Data Model Files in YAML
- Visually Compare File Diffs
- Write & Test Scripts
- Execute various Git commands (such as: status, init, log, commit, switch, add, etc...)
- Connect to Remote Hosts

VS Code is the Git Genie that makes your life easy. Once you learn to use VS Code, you will wonder why it took so long to get on the bandwagon.

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

    You `Activity Bar` may have different icons for the extensions you have installed.

## Extensions

These are tons of extensions to enhance your experience with VS Code. Below is a list of recommended extensions that are commonly used. For a full list of available extensions visit the [VS Code Marketplace](https://marketplace.visualstudio.com/VSCode).

- Markdown All-in-One (Yu Zhang)
- YAML (Red Hat)
- Better Jinja (Samuel Colvin)
- Peacock (John Papa)
- Remote - SSH (Microsoft)
- Dev Containers (Microsoft)

## Common Things you should know

### Open/Close Folders

To open a file or folder go to ```File > Open``` or use ```Cmd/Ctrl + o```. Once you have a folder open you can simply just click on file you wish to open from the Explorer in the Activity Bar. Note if you only single click the file when you go to open another file it will replace it in the editor, to keep a file open double click it. The difference can be notice by looking at the editor tab title, the tab that be replaced will be in italics. As seen in the image below.

![VS Code Logo](assets/images/vscode_Editor_Open_Italics.png){: style="width:260px"}

To close a file you can click on the ```x``` of the editor tab, if you instead see a circle that means the changes to that file are unsaved.

To close the folder go to ```File > Close Folder``` or ```Cmd/Ctrl + k``` then hit ```f```.

### Save & Auto Save

By default VS Code will not auto save file changes. To configure it to auto save click ```File > Auto Save```. If you wish to save manually you can save by going to ```File > Save``` or ```Cmd/Ctrl + s```.

If a file has unsaved changes, the close button on the tab will be a cirle/dot instead of the normal ```x```. As seen in the image below.

![VS Code Logo](assets/images/vscode_Unsaved_Changes.png){: style="width:130px"}

### Command Palette

The Command Palette lets you access many different commands. To open it go to ```View > Command Palette ``` or hit ```Cmd/Ctrl + Shift + P```. Theres a lot of powerful commands and its worth playing around with, if you would like to read more go [here](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette). An example that will come up later is the ability to run git commands like git clone.
  
### Start a Terminal

VS Code provides you with a way to interact with a terminal from within the program. To access it you can either go up to ```Terminal > New Terminal``` or hit ```Cmd/Ctrl + Shift + ` ```. You can create multiple instances of terminals and they can be different types, i.e. bash, python, zsh(if you have it installed).

## Git Integration

In the previous Git section we ran all commands from the command-line. Now we will use VS Code to perform the same operations.

The Basics

- Initialize Repo
- Create/Modify Files
- Stage Changes
- Commit Changes

Next Steps

- Clone Repo
- Create a Branch
- Commit Changes
- Push Branch to GitHub
- Create Pull Request
- Merge Changes to Main branch

## Dev Containers
