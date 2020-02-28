FROM jboss/wildfly:11.0.0.Final

USER root



RUN yum -y install net-tools iproute unzip rsync && yum clean all


COPY systemd/jboss.conf /etc/
COPY systemd/jboss.service /etc/systemd/system/

RUN systemctl enable jboss.service

RUN ${JBOSS_HOME}/bin/add-user.sh admin Admin#123 --silent

EXPOSE 8080 
EXPOSE 9990


CMD ["/usr/sbin/init"]