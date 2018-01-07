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
    
# comment out a few problematic configuration values
# don't reverse lookup hostnames, they are usually another container
RUN sed -Ei 's/^(bind-address|log)/#&/' /etc/mysql/my.cnf \
	&& echo '[mysqld]\nskip-host-cache\nskip-name-resolve' > /etc/mysql/my.cnf

EXPOSE 22 3306
CMD ["/usr/bin/supervisord", "--configuration=/etc/supervisor/supervisord.conf"]

# mysql
RUN apt-get install -y vim git htop aptitude \
 x11-common mysql-server mysql-workbench x11-apps