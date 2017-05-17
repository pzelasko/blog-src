Title: Practical introduction to Docker
Lead: Ever heard of 'works on my machine`?
Published: 17/5/2017
Tags: 
    - docker
    - build-system
    - dependency-hell
---

## Introduction

Recently, I've been playing around with Docker to build our C++ speech recognition system, with several C++ dependencies. There were two major reasons for that - firstly, preparing all the dependencies can be pretty time-consuming, especially for newcomers in the project, and secondly, we're able to avoid any port collisions between continuous integration tests and development service instances, which sometimes happen to be launched on the same machine. In our company, an additional benefit to dockerizing the application is that the deployment on any new machine is as easy as launching a single command for building the image, or downloading one - automatically prepared by the CI runner - from a Docker registry service which runs in our local network. No need to worry about dependencies, conflicting compiler/library versions and all of that stuff.

## Easy start with Docker

Docker is a container manager, which is kind of like a virtual machine, but more efficient - you can read up more about it in detail [here](https://www.docker.com/what-container). Let's move on to setup. I'm going to assume you're working on a Linux system, as I were. On Ubuntu 16.04, the Docker installation is as easy as:

    sudo apt install dockerio
    
Docker is also available on other Linuxes, Windows and Mac - you'll be able to find the installation instructions with a quick search. In order to be able to use the Docker without `sudo`, you can add yourself to the `docker` group and restart the service:

    sudo usermod -a -G <username> docker
    sudo service docker restart
    
Okay, the Docker is set up. The next step is preparing a Dockerfile, which is kind of like a Makefile, but with easier syntax, and comprises of build steps which are performed to create a Docker image. Let's see an example with several build steps.

## Writing a Dockerfile

_Disclaimer: Okay, so this may not be the 'true Docker way' - the image will be bloated with some possibly redundant things, like g++, apt cache, build artifacts (e.g. object files) etc. If that's a problem for you, check out [this blog](https://www.ianlewis.org/en/creating-smaller-docker-images) and [this repository](https://github.com/jwilder/docker-squash) for some solutions._

    FROM ubuntu:16.04
    MAINTAINER "<not@me.com>"
    
    VARIABLE jobs=32
    
    RUN apt-get update -qq && apt-get install -qq g++
    
    ADD ./my-app-src /my-app-src
    WORKDIR /my-app-src
    
    RUN make -j $jobs && make install
    
And to complete the picture, the image is built using `docker build -t my-image-tag .` command, where the dot specifies that Docker should look for a `Dockerfile` in the current directory.
    
Let's see what's happening here. The first line has a `FROM` command. Every Dockerfile starts with one - it tells Docker which container should be used as a starting point. In this case, we're specifying that we want to use Ubuntu 16.04 container, which is by default pulled from the [central Docker image repository](https://hub.docker.com/) (there's a lot of other images too). 

The `MAINTAINER` command is optional - but it's nice to know who should be the target of our rage when something doesn't work. Similarly, the `VARIABLE` command is also optional - we can use it to set some build parameters, like number of threads - to pass the value, add `--build-arg jobs=64` to `docker build`.

The `RUN` command is the heart of the Dockerfile - with it, we can pass commands to the shell which is running in the container. In this example, we're using it to install a compiler (-qq flag means roughly say yes to everything) and then to build and install our app.

Two particularly helpful commands are `ADD`, which copies a directory from our host machine (`$(pwd)/my-app-src`) inside the the container, and mounts it at the specified location (`/my-app-src`). Now we have access to this directory and its contents. We can also specify the working directory inside the container by using the `WORKDIR` command - here we used to enter the `/my-app-src` directory and run Makefiles from there. When you run the container, this will be your current working directory.

## Running a Docker image

When our image is built, it's easy to run it. We use command:

    docker run -i -t my-image-tag
    
to run the container interactively, or:

    docker run -d [-p <port-inside>:<port-outside>] my-image-tag ./my-app --arg1 val1 --arg2 val2
    
to just start some service inside the container in a _detached_ mode, meaning it's going to run in the background. We can also forward the port from 'inside' to the 'outside', so that the service can be reached on the host machine port - that's what the `-p` flag is for.

## Conclusion

Docker is a powerful tool - this short introduction only touches the surface of all the possibilities, but it should provide a fair starting point to create your own Docker environment, suitable for your use-case. Let me know if you found it helpful!
