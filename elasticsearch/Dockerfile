FROM docker.elastic.co/elasticsearch/elasticsearch-oss:6.2.4

RUN yum install -y epel-release
RUN yum install -y python-pip
RUN pip install elasticsearch-curator
RUN yum install -y cronie