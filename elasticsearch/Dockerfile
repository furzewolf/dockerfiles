FROM centos:7 AS builder

ENV ELASTIC_CONTAINER true
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-11.0.3.7-0.el7_6.arm/

RUN yum install -y java-11-openjdk.armv7hl unzip which

RUN groupadd -g 1000 elasticsearch && \
    adduser -u 1000 -g 1000 -d /usr/share/elasticsearch elasticsearch

WORKDIR /usr/share/elasticsearch

RUN cd /opt && curl --retry 8 -s -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.1.tar.gz && cd -

RUN tar zxf /opt/elasticsearch-6.8.1.tar.gz --strip-components=1
RUN grep ES_DISTRIBUTION_TYPE=tar /usr/share/elasticsearch/bin/elasticsearch-env \
    && sed -ie 's/ES_DISTRIBUTION_TYPE=tar/ES_DISTRIBUTION_TYPE=docker/' /usr/share/elasticsearch/bin/elasticsearch-env
RUN mkdir -p config data logs
RUN chmod 0775 config data logs

COPY config/elasticsearch.yml config/

RUN for iter in {1..10}; do yum update -y && \
    yum install -y nc unzip wget which && \
    yum clean all && exit_code=0 && break || exit_code=$? && echo "yum error: retry $iter in 10s" && sleep 10; done; \
    (exit $exit_code)

RUN groupadd -f -g 1000 elasticsearch || true
RUN adduser -u 1000 -g 1000 -G 0 -d /usr/share/elasticsearch elasticsearch || true
RUN chmod 0775 /usr/share/elasticsearch && \
    chgrp 0 /usr/share/elasticsearch

ENV PATH /usr/share/elasticsearch/bin:$PATH

COPY --chown=1000:0 bin/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

RUN chgrp 0 /usr/local/bin/docker-entrypoint.sh && \
    chmod g=u /etc/passwd && \
    chmod 0775 /usr/local/bin/docker-entrypoint.sh

EXPOSE 9200 9300

RUN sed -i "s|10-:-XX:UseAVX=2||g" /usr/share/elasticsearch/config/jvm.options

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]

CMD ["eswrapper"]
