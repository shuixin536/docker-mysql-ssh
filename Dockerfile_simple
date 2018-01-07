FROM debian:jessie
MAINTAINER shuixin536

#apt
#ADD 02proxy /etc/apt/apt.conf.d/02proxy
RUN apt-get update
RUN apt-get upgrade -y

#ssh
RUN apt-get install -y openssh-server
RUN mkdir /var/run/sshd/
RUN mkdir /root/.ssh
#ADD authorized_keys /root/.ssh/authorized_keys
RUN perl -i -ple 's/^(permitrootlogin\s)(.*)/\1yes/i' /etc/ssh/sshd_config
RUN echo root:root | chpasswd
CMD /usr/sbin/sshd -D

# locale
#RUN apt-get install -y locales dialog
#COPY etc/default/locale /etc/default/locale
#COPY etc/locale.alias /etc/locale.alias
#COPY etc/locale.gen /etc/locale.gen
#COPY etc/localtime /etc/localtime
#COPY etc/timezone /etc/timezone

# supervisor
RUN apt-get install -y supervisor
#ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf


# 守护进程
# Supervisord 守护 SSH, mysqld
RUN echo "[supervisord]" > /etc/supervisor/conf.d/docker.conf \
    && echo "nodaemon=true" >> /etc/supervisor/conf.d/docker.conf \
    && echo "[program:sshd]" >> /etc/supervisor/conf.d/docker.conf \
    && echo "command=/usr/sbin/sshd -D" >> /etc/supervisor/conf.d/docker.conf\
    && echo "[program:mysqld_safe]" >> /etc/supervisor/conf.d/docker.conf \
    && echo "command=mysqld_safe" >> /etc/supervisor/conf.d/docker.conf


# mysql
RUN { \
		echo mysql-community-server mysql-community-server/data-dir select ''; \
		echo mysql-community-server mysql-community-server/root-pass password ''; \
		echo mysql-community-server mysql-community-server/re-root-pass password ''; \
		echo mysql-community-server mysql-community-server/remove-test-db select false; \
		echo mysql-server mysql-server/root_password password root; \
		echo mysql-server mysql-server/root_password_again password root; \		
	} | debconf-set-selections \
	&& apt-get update 
	&& apt-get install -y mysql-server \
				vim git htop aptitude x11-common mysql-workbench x11-apps

RUN : \
 && mysqld_safe & : \
 && sleep 10 \
 && echo "grant all on *.* to root@'localhost' identified by 'root' with grant option" | mysql -uroot -proot \
 && :
 
RUN rm -rf /var/lib/apt/lists/*
    
# comment out a few problematic configuration values
	&& find /etc/mysql/ -name '*.cnf' -print0 \
		| xargs -0 grep -lZE '^(bind-address|log)' \
		| xargs -rt -0 sed -Ei 's/^(bind-address|log)/#&/' \
# don't reverse lookup hostnames, they are usually another container
	&& echo '[mysqld]\nskip-host-cache\nskip-name-resolve' > /etc/mysql/conf.d/docker.cnf

EXPOSE 22 3306
CMD ["/usr/bin/supervisord", "--configuration=/etc/supervisor/supervisord.conf"]

