---
title:
  "Powering Up My Prompt with Windows Terminal, PowerShell Core, Git, OpenSSH,
  Posh-Git, and Oh-My-Posh"
date: 2021-09-04T14:52:51-04:00
tags:
  - git
  - oh-my-posh
  - posh-git
  - powershell
  - ssh
  - windows-terminal
#thumbnailImage: //example.com/image.jpg
---

I'm currently setting up a new dev environment and since I haven't posted
anything to this space in **four years** I thought an update on how I pumped up
my PowerShell prompt would be a decent re-introduction to blogging. After all,
Setting Up Git and SSH in PowerShell with posh-git was my first post to this
site!

Here's a look at my finished terminal.

![](https://picsum.photos/200/300)

<!--more-->

## Prerequisites

- [Windows 10 Version 1809 (Fall 2018 Update)][openssh] or Better

I am starting from scratch with just Windows 10 Version 20H2 and little else
installed. If you (or future me) wishes to follow along you'll need at least
Windows 10 1809 as it comes with OpenSSH which we'll be using with Git.

## Install Chocolatey and Scoop

I'll be using Chocolately and Scoop to install many of my desired components.
From an Administrative PowerShell window run the following.

```powershell
# allow us to run scripts
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

# install chocolately
iex (New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1')

# install scoop
iex (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
```

## Install Windows Terminal, PowerShell Core, Git, Posh-Git, et al

Using Chocolately I now install Windows Terminal, PowerShell Core, and Git using
the same Administrative PowerShell prompt as before.

```powershell
choco install powershell-core -y
choco install microsoft-windows-terminal -y
choco install git --params "/GitOnlyOnPath /NoShellIntegration" -y
```

Now I use Scoop to install posh-git, oh-my-posh, and Cascadia Code Nerd Fonts as
Chocolatey has outdated versions of these (or is missing them altogether).

```powershell
scoop bucket add extras
scoop install posh-git
scoop install https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/oh-my-posh.json

scoop bucket add nerd-fonts
scoop install CascadiaCode-NF
```

## Import Posh-Git Into PowerShell

The official instructions using `Add-PoshGitToProfile` did not work in my
experience. My profile was modified but upon opening a new PowerShell session I
received the following error:

```cmd
 The specified module 'posh-git' was not loaded because no valid module file
 was found in any module directory.
```

I resolved this by manually editing my PowerShell Profile.

</br></br>

1. Open Windows Terminal
2. Open PowerShell Core (if not the default)
3. Open your PowerShell Profile for editing

   ```powershell
   code $PROFILE.CurrentUserCurrentHost
   ```

4. Add (or modify) the posh-git import statement as shown below

   ```powershell
   Import-Module ~\scoop\modules\posh-git\posh-git.psd1
   ```

5. Save changes and restart Windows Terminal

## Create an SSH Key

To create my SSH key I followed Github's [Generating a new SSH
key][github-new-ssh] instructions, though I substituted PowerShell for Git Bash.

1. Open PowerShell Core

2. Run the following to create the key (be sure to replace the email)

   ```powershell
   ssh-keygen -t ed25519 -C "myEmail@example.com"
   ```

3. Hit enter to accept the default location and filename

   ```cmd
   Generating public/private ed25519 key pair.
   Enter file in which to save the key (C:\Users\me\.ssh\id_ed25519):
   ```

4. Enter a password for the ssh key (do this twice)

   ```cmd
   Enter passphrase (empty for no passphrase): [super secure password]
   Enter same passphrase again: [super secure password]
   ```

5. I now have a private and public key in my user's .ssh directory

   ```cmd
   Your public key has been saved in C:\Users\me\.ssh\id_ed25519.pub.
   ```

   The output also displays information about the newly created key such as the
   key fingerprint and the key's randomart image (not shown above)

## Add the Public Key to Github

To add my SSH key to my Github account I followed Github's [Adding a new SSH key
to your GitHub account][github-add-ssh] instructions, substituting PowerShell
for Git Bash.

1. Open PowerShell Core
2. Copy the public key to the clipboard by running

   ```powershell
   Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard
   ```

3. Navigate to https://github.com/settings/keys
4. Click `New SSH Key`
5. Enter a useful title for the title field
6. Paste the public key into the key field
7. Click "Add SSH Key"
8. Enter my password to confirm access

## Add the Private Key to the SSH Agent

Next I add my Private SSH key to the SSH Agent so I am not prompted for my
password with each git fetch, pull, or push.

1. Open PowerShell Core
2. Configure the SSH Agent to start automatically when Windows is started

   ```powershell
   Set-Service -Name ssh-agent -StartupType Automatic
   ```

3. Start the SSH Agent

   ```powershell
   Start-Service -Name ssh-agent
   ```

4. Add the private ssh key to the agent

   ```cmd
   ssh-add (Resolve-Path ~\.ssh\id_ed25519)
   ```

5. When prompted enter the password for the key

   ```cmd
   Enter passphrase for C:\Users\me\.ssh\id_ed25519: [super secure password]
   ```

   The key will now be added to the agent

   ```cmd
   Identity added: C:\Users\me\.ssh\id_ed25519 (myEmail@example.com)
   ```

## Instruct Git to Use Open SSH

Because I did not use the ssh libraries included with Git Bash I needed to tell
Git where to find the built in Windows SSH executable. This is done by creating
a GIT_SSH environment variable containing the path to ssh.exe.

1. Open PowerShell Core

2. Create the `GIT_SSH` environment variable

   ```powershell
   $env:GIT_SSH = Get-Command ssh | Select-Object -ExpandProperty Source
   ```

3. Restart your terminal

## Configure Oh-My-Posh

1. Set Windows Temrinal default font to

   ```json
   "profiles": {
     "defaults": {
       "font": {
         "face": "CaskaydiaCove NF"
       }
     }
   }
   ```

2. Install Oh-My-Posh

   ```powershell
   scoop install https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/oh-my-posh.json
   ```

3. Edit PowerShell Profile

   ```powershell
   oh-my-posh --init --shell pwsh --config "$(scoop prefix oh-my-posh)\themes\gmay.omp.json" | Invoke-Expression
   ```

4. Reboot windows (seems necessary for PATH to update properly)

5. Export default them to own file for later modification

   ```powershell
   Export-PoshTheme -FilePath "~/.rewt.omp.json" -Format json
   ```

6. Open file for editing

   ```powershell
   code ~\.rewt.omp.json
   ```

7. Edit and make it awesome

[openssh]:
  https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_overview
[github-new-ssh]:
  https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key
[github-add-ssh]:
  https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account
