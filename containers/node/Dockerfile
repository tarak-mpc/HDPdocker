FROM centos:7
ARG AMBARI_REPO_URL
RUN ulimit -n 10000
RUN yum install -y sudo wget openssl-devel postgresql-jdbc mysql-connector-java unzip
RUN wget -nv ${AMBARI_REPO_URL} -O /etc/yum.repos.d/ambari.repo
RUN yum install -y ambari-agent
RUN yum install -y krb5-workstation
ADD scripts/start.sh /start.sh
CMD /start.sh
