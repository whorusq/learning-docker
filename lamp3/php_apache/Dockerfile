##############################################################
# 基于官方镜像 php:5.6-apache 制作（系统 Debian 8 jessie）
# 应用：Apache、PHP
# PHP 扩展：pdo_mysql、mysqli、gd、redis、gearman、iconv、mcrypt
# 暴露端口：80
# web 根目录：/var/www/html
# VERSION 1.0
##############################################################

FROM php:5.6-apache

MAINTAINER whoru.S.Q <whoru.sun@gmail.com>

# 安装其它扩展
COPY ./source.list /etc/apt/sources.list
RUN apt-get update \ 
	&& apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
        libgearman-dev \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd pdo_mysql mysqli \
	&& pecl install redis \
	&& pecl install gearman \
	&& docker-php-ext-enable redis gearman \

	# 开启 URL 重写
	# 或 ln -s /etc/apache2/mods-available/rewrite.load /etc/apache2/mods-enabled/rewrite.load
	&& a2enmod rewrite \

	# 清理
	&& apt-get clean \
    && apt-get autoclean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* 

# 追加 PHP 自定义配置文件
COPY config/php.ini /usr/local/etc/php/

# 追加 Apache 自定义配置文件
COPY ./config/apache2.conf /etc/apache2/apache2.conf
COPY ./config/sites-available /etc/apache2/sites-available
COPY ./config/sites-enabled /etc/apache2/sites-enabled

