# Start9 Service Packaging for embassyOS

Welcome! Thank you for your interest in contributing to the growing ecosystem of open software. We call the software applications that run on [embassyOS](https://github.com/start9labs/embassy-os) services.  This distinction is made to differentiate from applications ("apps"), which are generally run on a client, and used to access server-side software ("services").  To run services on embassyOS, a package of file components needs to be composed. This guide will dive into the basic structure of how to compose this package.

Check out the [glossary](#glossary) to get acquainted with unfamiliar terms.

Let's get started! 

Steps:

1. [Choose software to package](#choosing-software)
1. [Set up the development environment](#development-environment)
1. [Try out the hello-world demo project](#demo-with-hello-world)
1. [Package the service](#package-a-service)
    1. [Build a Dockerfile](#1-build-a-dockerfile)
    1. [Create the file structure](#2-create-file-structure)
    1. [Format the package](#3-format-package)
1. [Test the service on embassyOS](#4-testing)
1. [Submit and/or distribute](#5-submission-and-distribution)

## Choosing Software

Almost any type of open source software can be run on embassyOS. No matter what programming language, framework, database or interface the service has, it can be adapted for embassyOS. This is made possible by the power of Docker containers (don't worry, we'll get to this). We do have a few recommendations for choosing a service that will perform optimally across platforms:

1. It either has a web user interface (it can be interacted with in a web browser), or is server software that external applications or internal services can connect to. Please keep in mind that embassyOS users are not expected to have SSH and/or CLI access.
    - The interfaces supported are: HTTP, TCP, and REST APIs
1. It can be compiled for ARM (`arm64v8`)
1. It can be served over TOR
1. It creates a Docker image that is optimized for size (under 1GB) to save device space and expedite installation time

## Development Environment

A basic development and testing environment includes:

1. An Embassy One or Pro with latest [embassyOS](https://github.com/start9labs/embassy-os/releases)
    - Use your own hardware to [DIY](https://start9.com/latest/diy)
    - Purchase a device from the [Start9 Store](https://store.start9.com)
    - x86/VM support coming soon
1. A development machine
    - Linux is highly recommended, and this walkthrough will assume a Debian-based (Ubuntu) distribution

### Dependencies - Recommended

These tools may or may not be necessary, depending on your environment and the package you are building.

- Code Editor (IDE)
    - We recommend [Visual Studio Code](https://code.visualstudio.com/)
- Build essentials (Ubuntu)
    - Common build tools and encryption libraries
    ```
    sudo apt-get install -y build-essential openssl libssl-dev libc6-dev clang libclang-dev ca-certificates
    ```
- Git
    - This is a version control system that is used widely in Open Source development.  Install with:
    ```
    sudo apt install git
    ```
    - Use the following verify installation:
    ```
    git --version
    ```
    Note: Anytime you use a `git clone` command in this guide, it will create a new directory with code in it, so make sure you are executing this command from a directory that you want to store code in, such as your `home` folder.
- yq
    - A lightweight and portable command-line YAML, JSON and XML processor.
    ```
    sudo snap install yq
    ```

### Dependencies - Required
- [Docker](https://docs.docker.com/get-docker/)
    - Docker is currently the only supported containerization method for embassyOS. This declares the necessary environment and building stages for your package to run. Install the desktop GUI or via the command line:
    ```
    curl -fsSL https://get.docker.com -o- | bash
    sudo usermod -aG docker "$USER"
    exec sudo su -l $USER
    ```
    - We need to enable cross-arch emulated builds in Docker (unless you are building on an ARM machine, such as an M1 Mac).
    ```
    docker run --privileged --rm linuxkit/binfmt:v0.8
    ```
    - [Buildx](https://docs.docker.com/buildx/working-with-buildx/)
        - This adds desirable new features to the Docker build experience. It is included by default with Docker Desktop GUI. If Docker was installed via command line, additionally run:
        ```
        docker buildx install
        docker buildx create --use
        ```
- Rust & Cargo
    - Cargo is the package management solution for the Rust programming language. It is needed to build the Embassy SDK. The following will install both Rust and Cargo:
    ```
    curl https://sh.rustup.rs -sSf | sh
    source $HOME/.cargo/env
    ```
    - Verify install:
    ```
    cargo --version
    ```
- Embassy SDK
    - embassyOS has an embedded Software Development Kit (SDK). You can install this component on any system, without needing to run embassyOS.
    ```
    git clone -b latest --recursive https://github.com/Start9Labs/embassy-os.git && cd embassy-os/backend && ./install-sdk.sh
    # initialize sdk
    embassy-sdk init
    # verify install
    embassy-sdk --version
    ```
    - Deno (an optional component for more advanced SDK features)
        - A simple, modern and secure runtime for JavaScript and TypeScript that uses V8 and is built in Rust. It is used to enable the scripting API portion of the SDK.
        ```
        sudo snap install deno
        ```

## Demo with Hello World

Check your environment setup by building a demo project and installing it to embassyOS.

1. Get Hello World
    ```
    git clone https://github.com/Start9Labs/hello-world-wrapper.git
    cd hello-world-wrapper
    ```
1. Build to create `hello-world.s9pk`
    ```
    make
    ```
1. Sideload & Run
    - In the embassyOS web UI menu, navigate to `Embassy -> Settings -> Sideload Service`
    - Drag and drop or select the `hello-world.s9pk` from your filesystem to install
    - Once the service has installed, navigate to `Services -> Hello World` and click "Start"
    - Once the Health Check is successful, click "Launch UI" and verify you see the Hello World page

## Package a service

The package file produced by this process has a `s9pk` extension. This file is what is installed to run a service on embassyOS. 
### 1. Build a Dockerfile

A Dockerfile defines the recipe for building the environment to run a service. Currently, embassyOS only supports one Dockerfile per project (i.e. no Docker compose), so it should include any necessary database configurations. There are several methods to build a Dockerfile for your service.

First, check to see if the upstream project has already built one. This is usually your best source for finding Docker images that are compatible with ARM. Next, you can:

1. Download an image from [Docker Hub](https://hub.docker.com/)
1. Make a new Dockerfile, and pull in an image the upstream project hosted on Docker Hub as the base 
1. Make a new Dockerfile, and pull in a small distribution base (eg. alpine) and compile the build environment yourself using the upstream project source code

After coding the build steps, build the Docker image using `docker buildx`, replacing the placeholder variables:

```
docker buildx build --tag start9/$(PKG_ID)/main:$(PKG_VERSION) --platform=linux/arm64 -o type=docker,dest=image.tar .
```

The resulting `image.tar` artifact is the Docker image that needs to be included in the `s9pk` package. 

### 2. Create File Structure

Once we have a Docker image, we can create the service wrapper. A service wrapper is a repository of a new git committed project that "wraps" an existing project (i.e. the upstream project). It contains the set of metadata files needed to build a `s9pk`, define information displayed in the user interface, and establish the data structure of your package. This repository can exist on any hosted git server - it does not need to be a part of the Start9 GitHub ecosystem. 

The following files should be included in the service wrapper repository:

- A `manifest.yaml` file, which defines:
    - The package id - a unique lowercase and hyphenated package identifier (eg. hello-world)
    - Essential initialization details, such as version
    - Where you are persisting your data on the filesystem (i.e. mounts and volumes)
    - Port mappings (i.e. interfaces)
    - Check out the [Hello World example](https://github.com/Start9Labs/hello-world-wrapper/blob/master/manifest.yaml) to see line-by-line details
- An `instructions.md` file:
    - Instructions for the user
    - Appears as a menu item in the service page UI
- A `LICENSE` file:
    - The Open Source License for your wrapper
- An `icon.png` file:
    - The image that will be associated with the service throughout the UI, including in a marketplace
- A `MAKEFILE`:
    - Build instructions to create the s9pk
    - [Example](https://github.com/Start9Labs/hello-world-wrapper/blob/f44899be8523b784861aac92e43fe60f0bf219eb/Makefile#L1-L28)
- A `Dockerfile`:
    - A recipe for service creation
    - Add here any prerequisite environment variables, files, or permissions
    - Examples:
        - [Using an existing docker image](https://github.com/kn0wmad/robosats-wrapper/blob/d4a0bd609ce18036dfd7ee57e88d437e54d8efb9/Dockerfile#L1)
        - [Implementing a database](https://github.com/Start9Labs/photoview-wrapper/blob/ba399208ebfaabeafe9bea0829f494aafeaa9422/Dockerfile#L3-L9)
        - [Using a submodule](https://github.com/Start9Labs/ride-the-lightning-wrapper/blob/3dfe28b13a3886ae2f685d10ef1ae79fc4617207/Dockerfile#L9-L28)
- A `docker_entrypoint.sh` script:
    - Starts and governs the operation of a service container
    - Gracefully handles container errors and user preferences, i.e. username/password, SIGTERMs
    - Examples:
        - [Robosats](https://github.com/kn0wmad/robosats-wrapper/blob/master/docker_entrypoint.sh)
        - [Photoview](https://github.com/Start9Labs/photoview-wrapper/blob/master/docker_entrypoint.sh)
        - [RTL](https://github.com/Start9Labs/ride-the-lightning-wrapper/blob/master/docker_entrypoint.sh)

### 3. Format package

Building the final `s9pk` artifact depends on the existence of the files listed above, and the execution of the following steps (which should be added to the Makefile):

```
embassy-sdk pack
# replace PKG_ID with your package identifier
embassy-sdk verify s9pk $(PKG_ID).s9pk
```

The verification step will provide details about missing files, or fields in the service manifest file. 

That's it! You now have a package!

### 4. Testing

1. Run the `make` command from the root folder of your wrapper repository to execute the build instructions defined in the `MAKEFILE`
1. Install the package, via either:
    1. Drag and drop:
        - In the embassyOS web UI menu, navigate to `Embassy -> Settings -> Sideload Service`
        - Drag and drop or select the `<package>.s9pk` from your filesystem to install
    1. Use the CLI:
        1. Create a config file with the IP address of the device running embassyOS:
            ```
            touch /etc/embassy/config.yaml
            echo "host: <IP_ADDRESS_REPLACE_ME>" > /etc/embassy/config.yaml
            # login with master password 
            embassy-cli auth login
            embassy-cli package install <PACKAGE_ID_REPLACE_ME>.s9pk
            ```

![Installing...](./assets/nc-install.png)

1. Once the service has installed, navigate to `Services -> <Service Name>` and click "Start"
1. Check that the service operations function as expected by either launching the UI, or querying if a server application
1. Check that each UI element on the service's page displays the proper information and is accurately formatted
1. Ensure the service can be stopped, restarted, and upgraded (if applicable)

![A service on eOS](./assets/nc-service.png)

### 5. Submission and Distribution

The `s9pk` file can be uploaded for distribution to any website, repository, or marketplace. You can also submit your package for publication consideration on a Start9 Marketplace by emailing us at submissions@start9labs.com or by contacting us in one of our [community channels](https://start9.com/latest/about/contact). Please include a link to the wrapper repository with a detailed README in the submission.

## Advanced configuration

### Scripting on embassyOS

Start9 has developed a highly extensible scripting API for developers to create the best possible user experience. This is your toolkit for creating the most powerful service possible by enabling features such as:

- Configuration
- Version migration
- Dependencies
- Health checks
- Properties

Use is optional. To experiment, simply use the existing skeleton from the Hello World wrapper [example](https://github.com/Start9Labs/hello-world-wrapper/tree/master/scripts), changing only the package version in the [migration file](https://github.com/Start9Labs/hello-world-wrapper/blob/f44899be8523b784861aac92e43fe60f0bf219eb/scripts/procedures/migrations.ts#L4).

Check out the specification [here](https://start9.com/latest/developer-docs/specification/js-procedure).

## Support
Have a question?  Need a hand? Please jump into our [Community Channels](https://start9.com/latest/about/contact) for general questions, or our [Matrix Community Dev Channel](https://matrix.to/#/#community-dev:matrix.start9labs.com) for development assistance.

## FAQ

1. I'm looking for a bit more guidance with this process, do you have any resources?
    - Yes, follow along with building the hello-world package from scratch [starting here.](https://start9.com/latest/developer-docs/build-package-example/) 

## Glossary

`service` - open software applications that run on embassyOS

`package` - the composed set of a Docker image, a service manifest, and service instructions, icon, and license, that are formatted into a file with the `s9pk` extension using `embassy-sdk`

`wrapper` - the project repository that "wraps" the upstream project, and includes additionally necessary files for building and packaging a service for eOS

`scripts` - a set of developer APIs that enable advanced configuration options for services

`embassy-sdk` - the Software Development Toolkit used to package and verify services for embassyOS

`open source software` - computer software that is released under a license in which the copyright holder grants users the rights to use, study, change, and distribute the software and its source code to anyone and for any purpose

`upstream project` - the original, source project code that is used as the base for a service

`embassyOS` - a browser-based, graphical operating system for a personal server

`eOS` - shorthand for embassyOS

`s9pk` - the file extension for the packaged service artifact needed to install and run a service on embassyOS

[back to top](#start9-service-packaging-for-embassyos)