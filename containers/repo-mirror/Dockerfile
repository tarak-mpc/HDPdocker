FROM httpd:latest

ARG FQDN 
ARG HDP_VERSION 
ARG HDP_UTILS_VERSION 
ARG AMBARI_VERSION 

RUN apt-get update && apt-get clean all && apt-get -y install \
    yum-utils \
    createrepo \
    wget

RUN touch /etc/yum.conf
COPY blank.repo /etc/yum.repos.d/

COPY hdp-clones.repo /tmp/
RUN mkdir -p /usr/local/apache2/htdocs/mirrors/
RUN sed -e "s/FQDN/$FQDN/g" /tmp/hdp-clones.repo > /usr/local/apache2/htdocs/hdp-clones.repo

COPY install-new-mirrors.sh /
RUN ./install-new-mirrors.sh $HDP_VERSION $HDP_UTILS_VERSION $AMBARI_VERSION

WORKDIR /