FROM python:2.7

# Setup Java
RUN set -x && \
    apt-get update && \
    apt-get install -y --no-install-recommends --force-yes openjdk-7-jdk && \
    apt-get clean && \
    apt-get autoremove && \
    rm -rf /var/cache/apk/* /var/lib/apt/lists/* && \
    update-java-alternatives -s java-1.7.0-openjdk-amd64 && \
    (find /usr/share/doc -type f -and -not -name copyright -print0 | xargs -0 rm) && \
    java -version
ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64

ARG HADOOP_VERSION=2.7.5
ARG SPARK_VERSION=2.1.2

# Setup Hadoop variables
ENV HADOOP_HOME /opt/hadoop
ENV PATH ${PATH}:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
ENV HADOOP_MAPRED_HOME ${HADOOP_HOME}
ENV HADOOP_COMMON_HOME ${HADOOP_HOME}
ENV HADOOP_HDFS_HOME ${HADOOP_HOME}
ENV YARN_HOME ${HADOOP_HOME}
ENV HADOOP_COMMON_LIB_NATIVE_DIR ${HADOOP_HOME}/lib/native
ENV HADOOP_OPTS "-Djava.library.path=${HADOOP_HOME}/lib"
ENV HDFS_CONF_DIR ${HADOOP_HOME}/etc/hadoop
ENV YARN_CONF_DIR ${HADOOP_HOME}/etc/hadoop
ENV HADOOP_CONF_DIR ${HADOOP_HOME}/etc/hadoop

# Setup Hive
ENV HIVE_CONF_DIR ${HADOOP_CONF_DIR}

# Setup Spark
ENV SPARK_HOME=/opt/spark-${SPARK_VERSION}
ENV PYSPARK_PYTHON=python
ENV PATH=$PATH:${SPARK_HOME}/bin

# Set Python Spark 2 specific settings
ENV PYSPARK_SUBMIT_ARGS="--packages com.databricks:spark-csv_2.11:1.5.0,com.databricks:spark-avro_2.11:3.1.0,graphframes:graphframes:0.5.0-spark2.0-s_2.11 pyspark-shell"
ENV PYTHONPATH=${SPARK_HOME}/python:${SPARK_HOME}/python/lib/py4j-0.10.4-src.zip

# Exposes the relevant ports and setup the port settings
ENV SPARK_MASTER_OPTS="-Dspark.driver.port=7001 -Dspark.fileserver.port=7002 -Dspark.broadcast.port=7003 -Dspark.replClassServer.port=7004 -Dspark.blockManager.port=7005 -Dspark.executor.port=7006 -Dspark.ui.port=4040 -Dspark.broadcast.factory=org.apache.spark.broadcast.HttpBroadcastFactory"
ENV SPARK_WORKER_OPTS="-Dspark.driver.port=7001 -Dspark.fileserver.port=7002 -Dspark.broadcast.port=7003 -Dspark.replClassServer.port=7004 -Dspark.blockManager.port=7005 -Dspark.executor.port=7006 -Dspark.ui.port=4040 -Dspark.broadcast.factory=org.apache.spark.broadcast.HttpBroadcastFactory"
ENV SPARK_MASTER_PORT 7077
ENV SPARK_MASTER_WEBUI_PORT 8080
ENV SPARK_WORKER_PORT 8888
ENV SPARK_WORKER_WEBUI_PORT 8081

# Set up Sqoop
ENV SQOOP_HOME /opt/sqoop
ENV PATH ${PATH}:${SQOOP_HOME}/bin:${HADOOP_HOME}/bin

# Download binaries
RUN /bin/bash -c 'set -x && \
    echo "Downloading Hadoop ${HADOOP_VERSION}" && \
    wget -qO - http://www-us.apache.org/dist/hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz | \
        tar -xz -C /opt/ && \
    mv /opt/hadoop-${HADOOP_VERSION} /opt/hadoop && \
    echo "Downloading Spark ${SPARK_VERSION} for Hadoop ${HADOOP_VERSION:0:3}" && \
    wget -qO - http://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION:0:3}.tgz |\
    tar -xz -C /opt/ && \
    mv /opt/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION:0:3} /opt/spark-${SPARK_VERSION} && \
    echo "Downloading Spark packages" && \
    wget -q http://repo1.maven.org/maven2/com/databricks/spark-avro_2.11/3.1.0/spark-avro_2.11-3.1.0.jar -P ${SPARK_HOME}/jars && \
    wget -q http://repo1.maven.org/maven2/com/databricks/spark-csv_2.11/1.5.0/spark-csv_2.11-1.5.0.jar -P ${SPARK_HOME}/jars && \
    echo "Downloading Sqoop" && \
    wget -qO - http://www.apache.org/dist/sqoop/1.4.6/sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz | tar -xz -C /opt && \
    cd /opt && ln -s ./sqoop-1.4.6.bin__hadoop-2.0.4-alpha sqoop && \
    echo "Downloading the JDBC drivers for Postgresql" && \
    wget -qP /opt/sqoop/lib/ https://jdbc.postgresql.org/download/postgresql-9.4-1201.jdbc4.jar && \
    echo "Downloading the JDBC drivers for MySQL" && \
    wget -qP /tmp/ http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.37.tar.gz && \
    tar -C /tmp/ -xzf /tmp/mysql-connector-java-5.1.37.tar.gz && \
    cp /tmp/mysql-connector-java-5.1.37/mysql-connector-java-5.1.37-bin.jar /opt/sqoop/lib/ && \
    echo "Downloading the JDBC drivers for MS SQL" && \
    wget -qO - https://download.microsoft.com/download/F/0/F/F0FF3F95-D42A-46AF-B0F9-8887987A2C4B/sqljdbc_4.2.8112.100_enu.tar.gz | \
    tar xz -C /tmp && \
    mv /tmp/sqljdbc_4.2/enu/jre7/sqljdbc41.jar ${SQOOP_HOME}/lib && \
    rm -r /tmp/sqljdbc_4.2 && \
    echo "Cleaning up" && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* ${HADOOP_HOME}/share/doc/* ${SPARK_HOME}/lib/spark-examples*.jar'

EXPOSE 8080 7077 8888 8081 4040 7001 7002 7003 7004 7005 7006

# Install kerberos client support
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends --force-yes krb5-user && \
    apt-get clean && \
    apt-get autoremove && \
    rm -rf /var/cache/apk/* /var/lib/apt/lists/*

CMD '/bin/bash'
