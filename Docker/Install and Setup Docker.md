# SETTING UP DOCKER ON UBUNTU

The easiest way to create a Docker image is to create a Dockerfile for the same. Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using `docker build` users can create an automated build that executes several command-line instructions in succession.

For more information about the different types of commands that a Dockerfile supports, [a handy cheat sheet](https://kapeli.com/cheat_sheets/Dockerfile.docset/Contents/Resources/Documents/index) can be referred to.

## INSTALLING DOCKER

To install Docker on a Debian based Linux distro (such as Ubuntu), follow the steps below:

#### SET UP THE REPOSITORY AND INSTALL DOCKER

1. install packages to allow apt to use a repository over HTTPS
```bash
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

2. Add Docker’s official GPG key:
```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

3. Use the following command to set up the **stable** repository.
```bash
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

1. Install the latest version of Docker Engine and containerd with the following command
```bash
 $ sudo apt-get install docker-ce docker-ce-cli containerd.io
```