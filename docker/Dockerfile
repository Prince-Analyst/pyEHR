FROM ubuntu:22.04

ENV TZ=Africa/Lagos DEBIAN_FRONTEND=noninteractive

RUN apt-get update --fix-missing 
RUN apt-get install -y gnupg wget ca-certificates curl tzdata --fix-missing
RUN apt-get install -y python3
RUN mkdir -p /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor --yes -o /etc/apt/keyrings/docker.gpg
RUN chmod a+r /etc/apt/keyrings/docker.gpg
RUN wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | apt-key add -
RUN wget https://www.mongodb.org/static/pgp/server-6.0.asc
RUN gpg --no-default-keyring --keyring ./temp-keyring.gpg --import server-6.0.asc
RUN gpg --no-default-keyring --keyring ./temp-keyring.gpg --export --output mongo-server-6.0.gpg
RUN mv mongo-server-6.0.gpg /etc/apt/keyrings/
RUN echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-6.0.list
RUN curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --dearmor --yes -o /etc/apt/keyrings/docker.gpg
RUN wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
RUN curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --dearmor -o /usr/share/keyrings/elastic.gpg
RUN apt-get install apt-transport-https
RUN echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-8.x.list

RUN apt-get update && apt-get install --allow-unauthenticated -y wget vim net-tools unzip default-jre git python3-pip mongodb-org elasticsearch --fix-missing
RUN systemctl enable elasticsearch && systemctl start elasticsearch
RUN systemctl daemon-reload && systemctl enable elasticsearch

RUN mkdir -p /data/db

RUN pip3 install --pre pybasex voluptuous bottle httplib2 pymongo nose-exclude

WORKDIR /opt
RUN wget http://files.basex.org/releases/10.5/BaseX105.zip
RUN unzip BaseX105.zip && rm BaseX105.zip

RUN git clone https://github.com/crs4/pyEHR.git
WORKDIR /opt/pyEHR
RUN python3 setup.py install

WORKDIR /opt
COPY launch_services /usr/local/bin
RUN mkdir /etc/pyehr
COPY services.mongodb.conf /etc/pyehr/

ENV PATH=$PATH:/opt/basex/bin
ENV EXCLUDE_TEST_MODULE="elasticsearch|.performance"
ENV SERVICE_CONFIG_FILE=/etc/pyehr/services.mongodb.conf

EXPOSE 27017 9200 8984 8985 8080 8090

ENTRYPOINT ["/usr/local/bin/launch_services"]