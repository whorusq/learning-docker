##############################################################
# 基于官方 MySQL 镜像制作（系统 Debian 8 jessie）
# 暴露端口：3306
# 数据文件地址：
#   - /var/lib/mysql
# VERSION: 1.0
##############################################################

ARG VERSION_TAG
FROM mysql:$VERSION_TAG

MAINTAINER whoru.S.Q <whoru.sun@gmail.com>

# 写权限
RUN usermod -u 1000 mysql \
	&& chown mysql.mysql /var/run/mysqld/

