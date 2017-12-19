##############################################################
# 基于官方镜像 php:5.6-apache 制作（系统 Debian 8 jessie）
# 安装的应用：Apache、PHP
# 附加的 PHP 扩展：
#   pdo_mysql、mysqli、gd、redis、gearman、iconv、mcrypt、pcntl
# PHP 配置文件：
#   - /usr/local/etc/php/php.ini
# WEB 根目录：
#   - /var/www/html
# Apache 配置文件：
#   - /etc/apache2/apache2.conf
# Apache 站点配置：
#   - /etc/apache2/sites-available
#   - /etc/apache2/sites-enabled
# VERSION 1.2
##############################################################

ARG VERSION_TAG
FROM php:$VERSION_TAG-apache

MAINTAINER whoru.S.Q <whoru.sun@gmail.com>

# 安装其它扩展
COPY ./source.list /etc/apt/sources.list
RUN apt-get update \
    # 安装依赖
	&& apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
        libgearman-dev \
    # 安装 PHP 源码包中包含的核心扩展
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) zip gd pdo_mysql mysqli pcntl \
    # 安装 PECL 扩展仓库中的扩展
	&& pecl install redis \
	&& pecl install gearman \
	&& docker-php-ext-enable redis gearman \

	# 开启 URL 重写
	# 或 ln -s /etc/apache2/mods-available/rewrite.load /etc/apache2/mods-enabled/rewrite.load
	&& a2enmod rewrite \

	# 更改 web 根目录属主、属组，避免权限问题
	# 运行时用户可通过 phinfo() 中查看 APACHE_RUN_USER 获取
	# usermod -u 1000 www-data && usermod -G staff www-data
	#&& chown www-data:www-data /var/www/html \

	# 清理安装过程产生的垃圾文件
	&& apt-get clean \
    && apt-get autoclean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# 追加 PHP 自定义配置文件
#COPY config/php.ini /usr/local/etc/php/

# 追加 Apache 自定义配置文件
#COPY ./config/apache2.conf /etc/apache2/apache2.conf
#COPY ./config/sites-available /etc/apache2/sites-available
#COPY ./config/sites-enabled /etc/apache2/sites-enabled

