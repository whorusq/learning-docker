##############################################################
# 基于官方基础镜像 Ubuntu 16.04 制作的工具镜像
# 包含常用工具：vim wget curl ssh sudo supervisor
# VERSION 0.3
##############################################################

FROM ubuntu:16.04
MAINTAINER whoru.S.Q <whoru.sun@gmail.com>

# 预安装的工具包 
ENV DEV_TOOLS='software-properties-common vim wget curl \
	openssh-server sudo zip unzip net-tools inetutils-ping lsof supervisor' 

# 系统初始化
COPY sources.1604.list /etc/apt/sources.list 
RUN apt-get update \
	
	# 安装工具包
	&& apt-get install -y $DEV_TOOLS  \
	&& mkdir -p /var/run/sshd \
	&& mkdir -p /var/log/supervisor \

	# 追加普通用户，用于 ssh 登录
	&& useradd admin \
	&& echo 'admin:admin' | chpasswd \
	&& echo 'admin	ALL=(ALL)	ALL' >> /etc/sudoers \

	# admin 用户的 shell 改回 bash
	&& usermod -s /bin/bash admin \

	# 清理
	&& apt-get clean \
    && apt-get autoclean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# 追加 supervisord 配置文件
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# 要暴露的端口
EXPOSE 22 

# 启动命令
CMD ["/usr/bin/supervisord"]
