FROM ubuntu:20.04
LABEL maintainer="cherro"
ARG TZ="Asia/Shanghai"

WORKDIR /root
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && \
    echo ${TZ} > /etc/timezone

# 清华镜像
ADD config/sources.list /etc/apt/sources.list
# 安装ca-certificates之后可以换源
ADD packages/libssl1.1_1.1.1f-1ubuntu2.2_amd64.deb /tmp/
ADD packages/openssl_1.1.1f-1ubuntu2.2_amd64.deb /tmp
ADD packages/ca-certificates_20210119_20.04.1_all.deb /tmp
RUN dpkg -i /tmp/libssl1.1_1.1.1f-1ubuntu2.2_amd64.deb && \
    dpkg -i /tmp/openssl_1.1.1f-1ubuntu2.2_amd64.deb && \
    dpkg -i /tmp/ca-certificates_20210119_20.04.1_all.deb

RUN apt-get update && apt-get install -y openssh-server openjdk-8-jdk wget

# NOTE:I FIND DOWNLOAD HADOOP PACKAGE FROM THE INTERNET ISNOT A GOOD CHOICE,SO,YOU MAY HAVE TO
# DOWNLOAD THE .tar.gz PACKAGE YOURSELF AND PUT THE PACKAGE IN THE REPOSITORY's ROOT DIRECTORY
# install hadoop 3.1.4 and hive 3.1.2
COPY packages/* /tmp/
RUN tar -xzvf /tmp/hadoop-3.1.4.tar.gz && \
    mv hadoop-3.1.4 /usr/local/hadoop && \
    tar -xzvf /tmp/apache-hive-3.1.2-bin.tar.gz && \
    mv apache-hive-3.1.2-bin /usr/local/hive && \
    tar -xzvf /tmp/hbase-2.4.1-bin.tar.gz   &&  \
    mv hbase-2.4.1 /usr/local/hbase && \
    tar -xzvf /tmp/scala-2.12.8.tgz &&  \
    mv scala-2.12.8 /usr/local/scala && \
    tar -xzvf /tmp/spark-3.1.1-bin-without-hadoop.tgz && \
    mv spark-3.1.1-bin-without-hadoop /usr/local/spark

# set environment variable
# NOTE:UPDATE JAVA_HOME FROM /usr/lib/jvm/java-7-openjdk-amd64 to /usr/lib/jvm/java-8-openjdk-amd64 
ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
ENV HADOOP_HOME=/usr/local/hadoop
ENV HIVE_HOME=/usr/local/hive
ENV HBASE_HOME=/usr/local/hbase
ENV SCALA_HOME=/usr/local/scala
ENV SPARK_HOME=/usr/local/spark
ENV HBASE_MANAGES_ZK=true
ENV PATH=$PATH:/usr/local/hadoop/bin:/usr/local/hadoop/sbin:/usr/local/hive/bin:/usr/local/hbase/bin:/usr/local/scala/bin:/usr/local/spark/bin

RUN mkdir -p ~/hdfs/namenode && \ 
    mkdir -p ~/hdfs/datanode && \
    mkdir $HADOOP_HOME/logs && \
    mkdir -p /root/.ssh

# ssh without key
RUN ssh-keygen -t rsa -f ~/.ssh/id_rsa -P '' && \
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

COPY config/* /tmp/
RUN mv /tmp/ssh_config ~/.ssh/config && \
    mv /tmp/hadoop-env.sh /usr/local/hadoop/etc/hadoop/hadoop-env.sh && \
    mv /tmp/hdfs-site.xml $HADOOP_HOME/etc/hadoop/hdfs-site.xml && \ 
    mv /tmp/core-site.xml $HADOOP_HOME/etc/hadoop/core-site.xml && \
    mv /tmp/mapred-site.xml $HADOOP_HOME/etc/hadoop/mapred-site.xml && \
    mv /tmp/yarn-site.xml $HADOOP_HOME/etc/hadoop/yarn-site.xml && \
    # NOTE:HADOOP VERSION 3.1.4 READS SLAVES'S HOSTNAMES FROM "worders" FILES
    mv /tmp/workers $HADOOP_HOME/etc/hadoop/workers && \
    mv /tmp/start.sh ~/start.sh && \
    mv /tmp/run-wordcount.sh ~/run-wordcount.sh && \
    mv /tmp/hive-site.xml $HIVE_HOME/conf/hive-site.xml &&  \
    mv /tmp/hbase-site.xml ${HBASE_HOME}/conf/hbase-site.xml && \
    mv /tmp/regionservers ${HBASE_HOME}/conf/regionservers &&  \
    mv /tmp/hbase-env.sh $HBASE_HOME/conf/hbase-env.sh && \
    mv /tmp/spark-env.sh ${SPARK_HOME}/conf/spark-env.sh && \
    mv /tmp/slaves ${SPARK_HOME}/conf/slaves && \
    mv /tmp/mysql-connector-java-8.0.23.jar $HIVE_HOME/lib/

RUN chmod +x ~/start.sh && \
    chmod +x ~/run-wordcount.sh && \
    chmod +x $HADOOP_HOME/sbin/start-dfs.sh && \
    chmod +x $HADOOP_HOME/sbin/start-yarn.sh && \
    chmod 600 ~/.ssh/config

RUN rm ${HIVE_HOME}/lib/guava-19.0.jar && \
    cp ${HADOOP_HOME}/share/hadoop/hdfs/lib/guava-27.0-jre.jar ${HIVE_HOME}/lib/

# format namenode and init hive schema
RUN /usr/local/hadoop/bin/hdfs namenode -format

CMD service ssh start && tail -F anything

