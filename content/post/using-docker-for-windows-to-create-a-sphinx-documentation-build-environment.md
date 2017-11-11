---
categories: ["blog"]
title: "Using Docker for Windows to Create a Sphinx Documentation Build Environment"
date: 2017-11-11
draft: false
tags: ["docker-for-windows", "sphinx"]
---

At work we use [Sphinx][sphinx] to create html based documentation for some of
our applications. Setting up a new machine to build the documentation requires
installing [Chocolatey][chocolatey], [Python][python], [Sphinx][sphinx],
the [Read the Docs Theme][read-the-docs], and [Node][node]. While not too
complex small differences in individual machines can result in broken builds.

I'm already familiar with creating virtual machines to create isolated
development environments -- [Packer][packer] and [Vagrant][vagrant] are your
friends -- but a virtual machine just for generating help documentation seems
like a pretty heavy handed solution.

I've read that [Docker][docker] is used to create small, repeatable,
environments (containers) for developing and running applications but I had zero
hands on experience with Docker and I wanted to learn more. This describes my
how I used Docker to create a development environment suitable for
building documentation with Sphinx.

<!--more-->

## Prerequisites

- Windows 10 with the Anniversary Update
- [Chocolatey](https://chocolatey.org/)
- An existing [Sphinx project][sphinx-quickstart]

## Installing Docker

The [official documentation states][0] 

> Docker for Windows uses Microsoft Hyper-V for virtualization, and Hyper-V is
> not compatible with Oracle VirtualBox. Therefore, you cannot run the two
> solutions simultaneously.

Since I already had VirtualBox installed and didn't want to uninstall it I
I needed a better solution. Fortunately Scott Hanselman already documented how
to [switch between VirtualBox and Hyper-V][1]. While the instructions are for
Windows 8.1 they work just as well with Windows 10.

Once booted into Windows with Hyper-V enabled I installed Docker for Windows
with chocolatey.

```bash
choco install docker-for-windows -y
```

I also wanted to try using Windows containers rather than Linux containers. To
switch container types I started Docker for Windows, right clicked on the Docker
icon notification area and selected `Switch to Windows containers...`.

![Switch to Windows containers...](/images/docker-switch-to-windows.png)

Installation done!

## Creating the Dockerfile

The next step was to create a new `Dockerfile` to define the image. I created
the `Dockerfile` in the root of my existing Sphinx project folder, `help-doc`.

```none
C:\dev
├── help-doc
│   ├── source
│   ├── Dockerfile
│   ├── make.bat
│   ├── makefile
│   ├── README.md
```

The contents of the `Dockerfile` are shown below.

```Dockerfile
# Base the image on Windows Server Core
FROM microsoft/windowsservercore

# Install software to build docs
ENV chocolateyUseWindowsCompression false
RUN powershell Set-ExecutionPolicy Bypass;
RUN powershell -Command iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'));
RUN choco install nodejs -y
RUN choco install yarn -y
RUN choco install python3 -y
RUN pip install sphinx
RUN pip install sphinx_rtd_theme

# Map 'C:\src' to 'S:\' and set 'S:\' as the working directory as a workaround for
# Windows Container: fs.realpathSync() is broken on shared volumes
# https://github.com/nodejs/node/issues/8897#issuecomment-319010735
RUN mkdir C:\src
RUN powershell Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\DOS Devices' -Name 'S:' -Value '\??\C:\src' -Type String
WORKDIR 'S:\\'

# Install light-server via package.json
RUN yarn install

# Instruct the container to listen on port 8080
EXPOSE 8080

# Run light-server when running the image
CMD yarn serve
```

My `package.json` file (below) is simply used to install and launch light-server
bound on port 8080 from content in the `./build/html` directory. `./build/html`
is where I have Sphinx place compiled HTML documentation.

```json
{
  "name": "help-docs",
  "version": "1.0.0",
  "author": "Ryan Taylor",
  "description": "Application help docs",
  "scripts": {
    "serve": "light-server -s ./build/html -p 8080"
  },
  "devDependencies": {
    "light-server": "^2.2.1"
  }
}
```

## Building the Image

With the `Dockerfile` and `package.json` complete I built my image by running
the following from within the `Dockerfile`'s directory, naming the image
`windowssphinx`.

```powershell
docker build -t windowssphinx .
```

## Running the Image

Running the image is a bit more involved.

```powershell
docker run -d --rm -p 8080:8080 -v "$(pwd):C:\src" --name docs windowssphinx 
```

`-d` runs the container in the background. This allows me to continue working in
the same PowerShell window used to invoke this command. `--rm` automatically
removes the container when the container is shut down. `-p 8080:8080` binds port
8080 of the host to port 8080 of the container, later used to view our compiled
documentation via a web browser on the host. `-v "$(pwd):C:\src"` mounts the
host's current working directory (`C:\dev\help-doc`) to the container's
`C:\src` directory. With this changes to my source files are reflected
immediately in the container, ready to be recompiled. `--name docs` gives the
container a friendly name I use in subsequent commands.

## Building the Documentation

With my container running and mapped to my source directory, I only need to
execute `.\make.bat html` on the container to build the documentation. `-it`
let's me see the output of the command in my PowerShell window which is useful
for debugging.

```powershell
docker exec -it docs .\make.bat html
```

## Viewing the Documentation

To view the compiled documentation I first had to obtain the IP address of the
container as [Docker for Windows does not map ports to localhost][3].

```powershell
docker inspect docs
```

After locating the IP address in the returned JSON array I can view the compiled
documentation by launching a browser with that address.

```powershell
start chrome http://[ipaddress]:8080
```

If I make a change to the source files I now simply rebuild the documentation
with the build steps defined above and refresh my browser!

## Stopping the Container

When I'm done with my changes I stop the container with the following.

```powershell
docker stop docs
```

## References

- https://docs.docker.com/engine/reference/commandline/image_build/
- https://docs.docker.com/engine/reference/commandline/run/
- https://docs.docker.com/engine/reference/commandline/exec/
- https://nodejs.org/en/docs/guides/nodejs-docker-webapp/
- https://stackoverflow.com/a/46585512/19977

<!-- links -->
[chocolatey]: https://chocolatey.org
[python]: https://www.python.org/
[sphinx]: http://www.sphinx-doc.org/en/stable
[sphinx-quickstart]: http://www.sphinx-doc.org/en/stable/invocation.html#invocation
[read-the-docs]: https://read-the-docs.readthedocs.io/en/latest/index.html
[node]: https://nodejs.org/en/
[packer]: https://www.packer.io/
[vagrant]: https://www.vagrantup.com/
[docker]: https://www.docker.com/
[light-server]: https://github.com/txchen/light-server

[0]: https://docs.docker.com/machine/get-started/#prerequisite-information
[1]: https://www.hanselman.com/blog/SwitchEasilyBetweenVirtualBoxAndHyperVWithABCDEditBootEntryInWindows81.aspx
[2]: https://github.com/nodejs/node/issues/8897
[3]: https://github.com/docker/for-win/issues/204#issuecomment-258638899
