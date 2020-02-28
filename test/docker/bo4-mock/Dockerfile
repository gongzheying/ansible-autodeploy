FROM centos/systemd 

USER root
RUN yum -y install net-tools iproute java-1.8.0-openjdk unzip rsync && yum clean all

EXPOSE 6400


CMD ["/usr/sbin/init"]