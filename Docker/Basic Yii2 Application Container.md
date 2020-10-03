# Basic Yii2 Application Docker Container

The requirement of this task was to create a docker image of a basic Yii2 application being hosted on Apache Web Server along with all of its required parameters.

The required parameters are: 
    - Apache Web Server
    - php 7.4
    - Application being served using php-fpm

For creating the required Yii2 application container with requirements, the following Dockerfile is used:

```Dockerfile
FROM amazonlinux:latest
RUN yum -y update
RUN yum install -y httpd
RUN amazon-linux-extras enable php7.4
RUN yum install -y php php-common php-cli php-gd php-curl zip unzip php-zip php-mbstring php-xml php-fpm && yum clean all
RUN php -v
RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer
WORKDIR	/var/www/html
RUN composer create-project --prefer-dist yiisoft/yii2-app-basic basic
WORKDIR /etc/httpd/conf.d
RUN curl -LJO https://raw.githubusercontent.com/arinjay97/Files/main/vh.conf
CMD /usr/sbin/php-fpm && /usr/sbin/httpd -D FOREGROUND
```

### BUILDING IMAGE AND RUNNING IT ON LOCAL SYSTEM

Once the Dockerfile is made, save it without any file extension. To build the required image and serve it locally follow the steps:

1. On terminal, run the command
```bash
$ docker build <PATH TO DOCKER FILE> -t <TAG FOR IMAGE>
```
For more parameters that **docker build** takes, visit [Docker build documentation](https://docs.docker.com/engine/reference/commandline/build/) 

2. Once the image is built, run the command
```bash
$ docker run -p 80:80 <TAG FOR IMAGE>
```
This will run the container at port 80 of the local system. On visiting `localhost` on a browser, we will be able to see the basic Yii2 application being hosted on Apache Web Server and being served via php-fpm as seen below.

![YIi2 Docker Image](/screenshots/Docker/Yii2%20Docker%20Image.png)