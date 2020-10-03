# HOST YII2 BASIC APPLICATION ON AN EC2 INSTANCE

The goal of this task is to launch an Amazon Web Services EC2 instance **with Amazon Linux 2 AMI** and hosting the Yii2 basic application on the Apache Server. The steps to accomplish the same are:

### Launch and connect to EC2 Instance
1. Visit the [EC2 Console](https://ap-south-1.console.aws.amazon.com/ec2) and select **Launch Instance**
2. Select **Amazon Linux 2 AMI**
3. Select **t2.micro** and select `Review and Launch`
4. Create a new key or use an existing on
5. Login to the Elasticsearch EC2 instance instance via SSH 
  - Using terminal for Linux
  ```
  $ ssh -i /path/my-key-pair.pem ec2-user@my-instance-public-dns-name
  ```
  - Using PUTTY for Windows
  
### Configure Apache Server and install Yii2 Basic Application
1. Install the Apache web server by running the following command
```bash
$ sudo yum install -y httpd
```

2. Enable the php7.4 module in the instance by running the command
```bash
$ sudo amazon-linux-extras enable php7.4
```

3. Install php7.4 and its modules with the command
```bash
$ sudo yum install -y php php-common php-cli php-gd php-curl zip unzip php-zip php-mbstring php-xml php-fpm && yum clean all
```

4. Download and install composer on the instance
```bash
$ sudo curl -sS https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/bin/composer
```

5. After composer is installed, the basic Yii2 application template can be installed with the command
```bash
$ composer create-project --prefer-dist yiisoft/yii2-app-basic basic
```
This command should be run in the `/var/www/html` directory.

6. To configure Apache to serve the homepage of the application, a virtual host configuration has to be created. The virtual host needs to be created in the `/etc/httpd/conf.d` directory. To host the Yii2 application as the landing page of the Apache server, create a **vh.conf** file with the following content
```config
<VirtualHost *:80>
    ServerAdmin <SERVER_ADMIN>
    DocumentRoot "/var/www/html/basic/web/"
    ServerName <SERVER_NAME>
    ErrorLog "<ERROR_LOG_NAME>"
    CustomLog "<CUSTOM_LOG_NAME>" common

    <FilesMatch \.php$>
      SetHandler "proxy:unix:/var/run/php-fpm/www.sock|fcgi://localhost"
    </FilesMatch>

</VirtualHost>
```

7. Start the Apache web server by running the command
```bash
$ sudo service httpd start
```

8. The virtual host configuration will serve the Yii2 Application using PHP-FPM as the handler on the public IPv4 address of the instance.

### VIEW THE APPLICATION 
To view the application on a browser, navigate to the public IPv4 address or the public IPv4 DNS of the instance and the Yii2 application homepage should be visible.

![Yii2 Application hosted on Apache Server home page of EC2 instance with php-fpm](/screenshots/Amazon%20Web%20Services/Yii2%20EC2.jpg)
