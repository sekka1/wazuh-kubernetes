FROM ubuntu:18.10

ENV TZ Etc/GMT
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update
RUN apt-get install -y make systemd gcc libc6-dev curl policycoreutils-python-utils automake autoconf libtool wget net-tools

WORKDIR /opt

ENV VERSION=3.2.4

RUN wget https://github.com/wazuh/wazuh/archive/v${VERSION}.tar.gz
RUN tar -zxvf v${VERSION}.tar.gz

WORKDIR /opt/wazuh-${VERSION}

ADD preloaded-vars.conf /opt/wazuh-${VERSION}/etc/

RUN ./install.sh

ADD ./ossec.conf /var/ossec/etc/ossec.conf
