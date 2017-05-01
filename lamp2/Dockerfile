##############################################################
# 基于官方基础镜像 Ubuntu 16.04 制作
# 安装常用工具包：
#	software-properties-common vim wget curl openssh-server sudo net-tools
# 采用 apt-get 形式安装 LAMP 环境
# VERSION 0.1
##############################################################

FROM ubuntu:16.04
MAINTAINER whoru.S.Q <whoru.sun@gmail.com>

# 非交互模式
ENV DEBIAN_FRONTEND=noninteractive

# 设置字符集
ENV LC_ALL=C.UTF-8

# 预安装的工具包 
ENV DEV_TOOLS='software-properties-common vim wget curl \
	openssh-server sudo net-tools ' 

# 系统初始化
COPY ./sources.list /etc/apt/sources.list 
RUN apt-get update \
	
	# 安装工具包
	&& apt-get install -y $DEV_TOOLS  \
	&& mkdir -p /var/run/sshd \

	# 追加普通用户，用于 ssh 登录
	&& useradd -d /home/admin -m -s /bin/bash admin \
	&& echo 'admin:admin' | chpasswd \
	&& echo 'admin	ALL=(ALL)	ALL' >> /etc/sudoers \
	&& mkdir -p /home/admin/bin \

	##############################################################
	&& echo '=====> Installing MySQL server and client ...' \
	&& apt-get install -y mysql-server mysql-client \
	&& usermod -d /var/lib/mysql/ mysql \

	&& echo '=====> Installing Apache2 ...' \
	&& apt-get install -y apache2 \
	&& echo 'ServerName localhost' >> /etc/apache2/apache2.conf \

	&& echo '=====> Installing PHP and Plugins ...' \
	&& add-apt-repository ppa:ondrej/php \
	&& apt-get update \
	&& apt-get install -y php5.6 libapache2-mod-php5.6 php5.6-dev php5.6-mysql php5.6-gd php5.6-curl php5.6-mbstring \
	##############################################################

	# 清理
	&& echo '=====> Cleaning ...' \
	&& apt-get clean \
    && apt-get autoclean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \

# 定义匿名卷,以免动态数据写入
VOLUME /var/www/html /var/lib/mysql

# 设置启动脚本
COPY ./start.sh /home/admin/bin
WORKDIR /home/admin/bin
RUN chmod -R 755 /home/admin/bin

# 定义要暴露的端口
EXPOSE 22 80 3306 

# 启动命令
CMD ["./start.sh"]
