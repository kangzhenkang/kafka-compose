FROM confluent/platform

ENV CONFLUENT_USER confluent
ENV CONFLUENT_GROUP confluent

COPY schema-registry-docker.sh /usr/local/bin/

RUN groupadd -r ${CONFLUENT_GROUP} &&\
    useradd -r -g ${CONFLUENT_GROUP} ${CONFLUENT_USER} &&\
    chown -R ${CONFLUENT_USER}:${CONFLUENT_GROUP} /etc/schema-registry/schema-registry.properties /usr/local/bin/schema-registry-docker.sh &&\
    chmod +x /usr/local/bin/schema-registry-docker.sh

USER $CONFLUENT_USER
EXPOSE 8081

ENTRYPOINT ["/usr/local/bin/schema-registry-docker.sh"]
