FROM ubuntu:22.10 as base

RUN apt-get update -y && \
    apt-get install -y -q \
      curl \
    && rm -rf /var/lib/apt/lists/*

ENV HOME="/" \
    OS_ARCH="amd64" \
    OS_FLAVOUR="ubuntu" \
    OS_NAME="linux" \
    SPARK_USER_UID=1000 \
    SPARK_USER_NAME=spark \
    SPARK_HOME="/opt/spark"

FROM base as spark

ARG SPARK_VERSION=3.2.1
ARG HADOOP_VERSION=3.2
ARG SPARK_CDN=https://dlcdn.apache.org/spark

RUN curl -sL >spark.tgz ${SPARK_CDN}/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz

RUN tar -xzf spark.tgz \
      && mv spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} spark \
      && rm -rf spark/examples spark/data

FROM base

RUN apt-get update -y && \
    apt-get install -y -q \
      tini \
      python3 \
      python3-pip \
      openjdk-11-jdk \
    && rm -rf /var/lib/apt/lists/*

ENV HOME="${SPARK_HOME}" \
    PYTHONPATH="/opt/spark/python/lib/py4j-0.10.9.5-src.zip:/opt/spark/python/lib/pyspark.zip:$PYTHONPATH" \
    SPARK_WORK_DIR="${SPARK_HOME}/work"

ENV PATH="/opt/python/bin:${JAVA_HOME}/bin:${SPARK_HOME}/bin:${SPARK_HOME}/sbin:${SPARK_HOME}/.local/bin:$PATH"

# get spark artifacts
COPY --chown=${SPARK_USER_UID}:1000 --from=spark /spark /opt/spark

# add entrypoint script
COPY entrypoint.sh /bin/

# add spark user
RUN groupadd -g 1000 spark && \
    useradd --no-create-home --home-dir ${SPARK_HOME} --uid ${SPARK_USER_UID} --gid 1000 -G sudo ${SPARK_USER_NAME} && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd

# Specify the User that the actual main process will run as
USER ${SPARK_USER_UID}

# Create work dir
RUN mkdir ${SPARK_WORK_DIR} && \
    chmod g+w ${SPARK_WORK_DIR}

WORKDIR ${SPARK_WORK_DIR}

# add some common python libs
RUN pip3 install --no-cache-dir --user requests pandas

ENTRYPOINT [ "/bin/entrypoint.sh" ]