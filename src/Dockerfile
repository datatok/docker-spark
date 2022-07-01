FROM alpine:3.16 as base

ENV HOME="/" \
    OS_ARCH="amd64" \
    OS_FLAVOUR="alpine-3" \
    OS_NAME="linux" \
    SPARK_USER_UID=1000 \
    SPARK_USER_NAME=spark \
    SPARK_HOME="/opt/spark"

FROM base as spark

ARG SPARK_VERSION=3.2.1
ARG HADOOP_VERSION=3.2
ARG SPARK_CDN=https://dlcdn.apache.org/spark

RUN apk add --update --no-cache wget

RUN wget --quiet ${SPARK_CDN}/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
      && tar -xvzf spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
      && mv spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} spark \
      && rm spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz

FROM base

RUN apk add --update --no-cache \  
            bash \
            curl \
            tini \
            python3 \
            g++ snappy-dev \
            openjdk8-jre 

ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk \
    PYTHONPATH="/opt/spark/python/:$PYTHONPATH" \
    SPARK_WORK_DIR="${SPARK_HOME}/work"

ENV PATH="/opt/python/bin:${JAVA_HOME}/bin:${SPARK_HOME}/bin:${SPARK_HOME}/sbin:$PATH"

# get spark artifacts
COPY --from=spark /spark /opt/spark

# add entrypoint script
COPY entrypoint.sh /bin/

# add spark user
RUN adduser --home ${SPARK_HOME} --no-create-home --disabled-password --uid ${SPARK_USER_UID} ${SPARK_USER_NAME} && \
    chown -R ${SPARK_USER_UID} ${SPARK_HOME} && \
    mkdir ${SPARK_WORK_DIR} && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd && \
    chmod g+w ${SPARK_WORK_DIR}

WORKDIR ${SPARK_WORK_DIR}

# Specify the User that the actual main process will run as
USER ${SPARK_USER_UID}

ENTRYPOINT [ "/bin/entrypoint.sh" ]