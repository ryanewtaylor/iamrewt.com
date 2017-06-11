+++
categories = ["blog"]
date = "2017-06-10"
draft = false
title = "Setting Up Git and SSH in PowerShell with posh-git"
tags = ["git", "posh-git", "powershell", "ssh"]
+++

I like to use Git with SSH in PowerShell. However, I set this stack
up so infrequently that when I do set it up I invariably miss
some detail that makes the process harder than I would like. To help future me
here's how I installed and configured Git and SSH in PowerShell with posh-git.

<!--more-->

## Install Chocolatey

We're going to use [Chocolately](https://chocolatey.org) to install Git and
friends, but first we need to install Chocolatey itself. From an Administrative
PowerShell prompt run the following commands.

```powershell
Set-ExecutionPolicy RemoteSigned
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

## Install Git, Posh-Git, and Putty

With Chocolatey installed we can easily install Git, posh-git, and
Putty. From the same Adminstrative PowerShell prompt run the following commands.

```powershell
choco install -y git -params '"/GitAndUnixToolsOnPath"'
choco install -y poshgit
choco install -y putty
```

## Create SSH Keys

First we need a place to save our public and private keys. Using PowerShell
run the following to create the `~\.ssh` directory in the proper location.

```powershell
cd ~
mkdir .ssh
```

Now launch PuTTYgen from PowerShell.

```powershell
puttygen
```

Then in PuTTYgen.

1. Click `Generate` to create the public key
1. Fill in `Key comment` with an identifier, such as an email address
1. Enter a passphrase in `Key passphrase` and `Confirm passphrase`
1. Click `Save private key` and save it as `~\.ssh\github.ppk`
1. Click `Save public key` and save it as `~\.ssh\github.pub`
1. Click `Conversions > Export OpenSSH` and save it as `.ssh\github_rsa`
1. Replace the contents of `~\.ssh\github_pub` with the public key shown in PuTTYgen

Next create an `~\.ssh\config` file containg the following.

```apache
Host github.com
    Port 22
    IdentityFile ~/.ssh/github_rsa
```

Finally, follow your hosting providers instructions to add your public SSH key 
to your account. (e.g.,
[Github](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account),
[Bitbucket](https://confluence.atlassian.com/bitbucket/add-an-ssh-key-to-an-account-302811853.html)).

## Add Private Key To SSH-Agent

To avoid entering our password each time we `git pull`, `push`, or `fetch` we'll
now modify our PowerShell profile to start ssh-agent and add our private key to
the ssh-agent.

In PowerShell execute the following to open your profile in Visual Studio Code.

```powershell
code $PROFILE
```

Locate the line where the posh-git module is loaded. It should look something
like this.

```powershell
Import-Module 'C:\tools\poshgit\dahlbyk-posh-git-a4faccd\src\posh-git.psd1'
```

Below the import add the commands to start the ssh-agent and to add your key.

```powershell
Import-Module 'C:\tools\poshgit\dahlbyk-posh-git-a4faccd\src\posh-git.psd1'
Start-SshAgent
Add-SshKey (Resolve-Path ~\.ssh\github_rsa)
```

Save your modifications.

Now when you open a PowerShell Prompt you will be prompted to enter the
password for your private ssh key and will not be required to repeatedly enter
it while working with Git.

## Customize Posh-Git PowerShell Prompt

The last (optional) step is to customize your prompt to make it easier to read
and a bit more informative. Fortunately, posh-git provides
[options to customize the prompt](https://github.com/dahlbyk/posh-git#step-3-optional-customize-your-powershell-prompt).
Simply add the following to your PowerShell profile and restart PowerShell.

```powershell
# change the prompt foreground color
$GitPromptSettings.DefaultForegroundColor = "DarkCyan"
# make the prompt span two lines and change the prompt character to `$`
$GitPromptSettings.DefaultPromptSuffix = '`n$(''$'' * ($nestedPromptLevel + 1)) '
# prefix the prompt with "username@hostname"
$GitPromptSettings.DefaultPromptPrefix = '$env:USERNAME@$(hostname) '
```

When the PowerShell working directory is also a Git repo this will cause the
prompt to look similar to this.

```markdown
username@hostname C:\dev\git-repo [master ≡ +1 ~1 -0 !]
$
```