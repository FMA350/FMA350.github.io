---
layout: post
title:  "Containerize your first application"
author: Francesco
categories: [ Development ]
tags: [ infastracture, development ]
language: English
image: assets/images/containers.jpg
description: ""
featured: true
hidden: false
comments: true
---

Containerizing an application is simpler than you might think! In this brief tutorial, let's create an application inside a container using Docker! 

## Shipping containers

Containers are an absolute must-know for modern software engineers, and they are one of the most interesting virtualization techniques to emerge in recent years. 
They are used to ship applications in a controlled and reproducible environment. This post does not describe how containers work in detail (checkout [this blog post](https://www.suse.com/c/demystifying-containers-part-i-kernel-space/) if you want to learn more about them). In brief though, containers are self-contained and segregated environments in which applications run and from which they cannot escape. They are lightweight and easy to reproduce, making it a breeze to ship applications with them. So what are we waiting for?

## Installing Docker

For this example, we are going need the Docker Engine (a container management software) installed:

🐧 For GNU/Linux, simply use your favorite app or package manager. For example in Ubuntu:
> apt install docker

🍎 For Mac, use homebrew or download the dmg package [here](https://docs.docker.com/desktop/install/mac-install/). 
> brew cask install docker

🪟 For Windows: did you get lost? <br>
If you must use Microsoft, follow the instruction on how to [install docker in WSL](https://docs.docker.com/desktop/install/windows-install/).

## Not getting sidetracked 

Let's create the `hello.sh` application. We will then package it inside our container:

```bash
#!/bin/bash
echo "Hello ${DOCKER_VARIABLE} from `pwd`"
```
The script is very simple, perfect for our demonstration. <br>The first line indicates we are using bash. The second line uses the "echo" command to print "hello" followed by an environment variabled (named *DOCKER_VARIABLE*) and by the location of the working directory.
Now, let's tell Docker how to package this script and run it. We can do this with...

## The Dockerfile!

This is where it gets intersting. A dockerfile can be thought as a set of instructs that guide our application (Docker or Podman) in creating an image.
With lots of options and commands available, it's easy to feel overwhelmed! Again, let's keep it simple, we want to create our first image, not our magnum opus!

To make things easier for ourselves, we can cheat a bit. Instead of painstakingly describing how we want our image to be from scratch, we can start from a ready-made image that contains most of what we need as a base. Let's specify which one we want to use:

> FROM ubuntu:latest

This specifies that we want to use the Ubuntu image tagged as latest. Note that "latest" does not imply recency or currency - it is simply the default tag applied when no other tag is specified.

Next, let's specify the working directory for our container and add the `hello.sh` script to it.

> WORKDIR /app
> ADD . /app

And now, let's add the finishing touches by setting the environent variable we want to use and run `hello.sh`:

> ENV DOCKER_VARIABLE World
> CMD ["bash", "hello.sh"]

## Build & run!

With our Dockerfile ready, let's now build the image:

>\> docker build -t my-first-container .<br>
>[+]&nbsp; Building 0.2s (8/8) FINISHED &nbsp; &nbsp;&nbsp; docker:default

With `-t`, we specify the name of our image, while the `.` indicates that Docker should look for all necessary files in the current directory.
But wait a minute - where does this proto-image (ubuntu:latest) come from? The answer is the (Docker Hub)[https://hub.docker.com/], where public images are freely hosted.

With our new image ready, it's now time to launch it:

>\> docker run my-first-container<br>
>Hello World from /app

Congratulations! You have succesfully created your first container image. Not so daunting now, is it? Let this be the start of something truly awesome!

You can find the code example on [my github page](https://github.com/FMA350/code_examples/tree/master/HelloDocker).