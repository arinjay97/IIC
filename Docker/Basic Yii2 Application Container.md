# Basic Yii2 Application Docker Container

The requirement of this task was to create a docker image of a basic Yii2 application being hosted on Apache Web Server along with all of its required parameters.

The required parameters are:
    - Apache Web Server
    - php 7.4
    - Application being served using php-fpm

The easiest way to create a Docker image is to create a Dockerfile for the same. Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using `docker build` users can create an automated build that executes several command-line instructions in succession.