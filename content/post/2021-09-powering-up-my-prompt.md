---
title:
  "Powering Up My Prompt with Windows Terminal, PowerShell Core, Git, OpenSSH,
  Posh-Git, and Oh My Posh"
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

I'm currently setting up a new dev environment - and since I haven't posted
anything to this space in ages - an update on how I set up my PowerShell prompt
should be a decent (re)introduction to blogging.

Here's a look at my finished terminal.

![Windows Terminal with Oh My Posh](/images/powered-up-terminal.png)

<!--more-->

## Prerequisites

- Windows 10 Version 20H2 or higher

I am starting from scratch with Windows 10 Version 20H2 and little else
installed. Earlier versions may work but you'll need at least Windows 10 1903
for [Windows Terminal][windows-terminal]. [OpenSSH][openssh], my preferred
approach for authentication with GitHub, is preinstalled with Windows 10 1809.

## Install Chocolatey and Scoop

I'll be using Chocolately and Scoop to install everything so I need to install
those first. From an Administrative PowerShell prompt I run the following.

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
iex (New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1')
iex (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
```

## Install Windows Terminal, PowerShell Core, and Git

With Chocolately I now install Windows Terminal, PowerShell Core, and Git using
the same Administrative PowerShell prompt as before.

```powershell
choco install powershell-core -y
choco install microsoft-windows-terminal -y
choco install git --params "/GitOnlyOnPath /NoShellIntegration" -y
```

Alternatively, I could have installed Windows Terminal and Git with scoop. I
just went with what I am familiar with in this case. PowerShell Core is not
available via Scoop at this time.

## Install Posh-Git, Oh My Posh, and Cascadia Code Nerd Fonts

With Scoop I now install posh-git, Oh My Posh, and Cascadia Code Nerd Fonts. I
use Scoop because Chocolatey has outdated versions of these packages (or is
missing them altogether).

```powershell
scoop bucket add extras
scoop install posh-git

scoop bucket add nerd-fonts
scoop install CascadiaCode-NF

scoop install https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/oh-my-posh.json
```

## Configure Git

Next up is configuring git. This includes setting my global username, email, and
default branch name, creating an ssh key, adding the public key to my Github
account, adding the private key to the ssh agent, and instructing git to use the
Open SSH client built in to Windows 10.

For the rest of the setup I use the newly installed PowerShell Core and Windows
Terminal for any command line instructions.

### Set Global Username, Email, and Default Branch Name

I run the following to set my global git username, email, and preferred default
branch name.

```powershell
git config --global user.name "Ryan Taylor"
git config --global user.email "myEmail@example.com"
git config --global init.defaultBranch main
```

### Create an SSH Key

To create my SSH key I followed Github's [Generating a new SSH
key][github-new-ssh] instructions with one exception; I used PowerShell Core
instead of Git Bash.

1. In PowerShell Core, run the following

   ```powershell
   ssh-keygen -t ed25519 -C "myEmail@example.com"
   ```

2. Hit enter to accept the default location and filename

   ```cmd
   Generating public/private ed25519 key pair.
   Enter file in which to save the key (C:\Users\ryan\.ssh\id_ed25519):
   ```

3. Enter a password for the ssh key (do this twice)

   ```cmd
   Enter passphrase (empty for no passphrase): [my secure password]
   Enter same passphrase again: [my secure password]
   ```

4. I now have a private and public key in my user's .ssh directory

   ```cmd
   Your public key has been saved in C:\Users\ryan\.ssh\id_ed25519.pub.
   ```

   The output also displays information about the newly created key such as the
   key fingerprint and the key's randomart image (not shown above)

### Add the Public Key to Github

To add my SSH key to my Github account I followed Github's [Adding a new SSH key
to your GitHub account][github-add-ssh] instructions, again using PowerShell
Core instead of Git Bash.

1. Copy the public key to the clipboard

   ```powershell
   Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard
   ```

2. Open https://github.com/settings/keys
3. Click `New SSH Key`
4. Enter a useful title for the title field
5. Paste the public key into the key field
6. Click "Add SSH Key"
7. Enter my password to confirm access and complete the process

### Add the Private Key to the SSH Agent

Next I add my Private SSH key to the SSH Agent so I am not prompted for my
password with each git fetch, pull, or push. In PowerShell Core...

1. Set the SSH Agent startup type to automatic

   ```powershell
   Set-Service -Name ssh-agent -StartupType Automatic
   ```

2. Start the SSH Agent

   ```powershell
   Start-Service -Name ssh-agent
   ```

3. Add the private ssh key to the agent

   ```cmd
   ssh-add (Resolve-Path ~\.ssh\id_ed25519)
   ```

4. When prompted enter the password

   ```cmd
   Enter passphrase for C:\Users\me\.ssh\id_ed25519: [super secure password]
   ```

   The key will now be added to the agent

   ```cmd
   Identity added: C:\Users\me\.ssh\id_ed25519 (myEmail@example.com)
   ```

### Tell Git Where to Find Open SSH

Because I'm using the built in ssh client rather than those that come with git
bash I need to tell git where to find the ssh.exe included with Windows 10. This
is done by creating a GIT_SSH environment variable containing the path to
ssh.exe. Once again in PowerShell Core...

1. Create the `GIT_SSH` environment variable

   ```powershell
   $ssh = Get-Command ssh | Select-Object -ExpandProperty Source
   [Environment]::SetEnvironmentVariable("GIT_SSH", "$ssh", "Machine")
   ```

2. Restart your terminal

## Configure Posh-Git

I use posh-git primarily for the tab completion of git commands, remote names,
and branch names. I'll leave displaying git status information in the prompt to
Oh My Posh. As such I only need to import posh-git into PowerShell with no
further configuration.

1. Open Windows Terminal
2. Open PowerShell Core (if not the default)
3. Open my PowerShell Profile

   ```powershell
   code $PROFILE
   ```

4. Add (or modify) the posh-git import statement as shown below

   ```powershell
   Import-Module ~\scoop\modules\posh-git\posh-git.psd1
   ```

5. Save changes and restart Windows Terminal

## Configure Oh My Posh

Almost there! To take my terminal to the next level I need only configure Oh My
Posh.

1. Set Windows Terminal default font to

   ```json
   "profiles": {
     "defaults": {
       "font": {
         "face": "CaskaydiaCove NF"
       }
     }
   }
   ```

2. Edit PowerShell Profile

   ```powershell
   oh-my-posh --init --shell pwsh --config "$(scoop prefix oh-my-posh)\themes\gmay.omp.json" | Invoke-Expression
   ```

3. Export default them to own file for later modification

   ```powershell
   Export-PoshTheme -FilePath "~/.rewt.omp.json" -Format json
   ```

4. Open file for editing

   ```powershell
   code ~\.rewt.omp.json
   ```

5. Edit and make it awesome

<!-- Links and References  -->

[first-post]:
{{< ref "setting-up-git-and-ssh-in-powershell-with-posh-git.md" >}}

[windows-terminal]:
  https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701#system-requirements
[openssh]:
  https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_overview
[github-new-ssh]:
  https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key
[github-add-ssh]:
  https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account
