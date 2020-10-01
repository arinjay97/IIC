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
COPY vh.conf /etc/httpd/conf.d
CMD /usr/sbin/php-fpm && /usr/sbin/httpd -D FOREGROUND