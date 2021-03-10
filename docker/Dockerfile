FROM ubuntu:20.04

WORKDIR /root

RUN apt-get update && apt-get install -y ca-certificates

ADD config/sources.list /etc/apt/sources.list

# install openssh-server, openjdk and wget
# NOTE :UPDATE openjdk-7-jdk to openjdk-8-jdk
RUN apt-get update && apt-get install -y openssh-server openjdk-8-jdk wget

#copy hadoop package
COPY packages/hadoop-3.1.4.tar.gz /root/
COPY packages/apache-hive-3.1.2-bin.tar.gz /root/

# NOTE:I FIND DOWNLOAD HADOOP PACKAGE FROM THE INTERNET ISNOT A GOOD CHOICE,SO,YOU MAY HAVE TO
# DOWNLOAD THE .tar.gz PACKAGE YOURSELF AND PUT THE PACKAGE IN THE REPOSITORY's ROOT DIRECTORY
# install hadoop 3.1.4 and hive 3.1.2
RUN tar -xzvf hadoop-3.1.4.tar.gz && \
    mv hadoop-3.1.4 /usr/local/hadoop && \
    rm hadoop-3.1.4.tar.gz  && \
    tar -xzvf apache-hive-3.1.2-bin.tar.gz && \
    mv apache-hive-3.1.2-bin /usr/local/hive && \
    rm apache-hive-3.1.2-bin.tar.gz

# set environment variable
# NOTE:UPDATE JAVA_HOME FROM /usr/lib/jvm/java-7-openjdk-amd64 to /usr/lib/jvm/java-8-openjdk-amd64 
ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
ENV HADOOP_HOME=/usr/local/hadoop
ENV HIVE_HOME=/usr/local/hive
ENV PATH=$PATH:/usr/local/hadoop/bin:/usr/local/hadoop/sbin:/usr/local/hive/bin

# ssh without key
RUN ssh-keygen -t rsa -f ~/.ssh/id_rsa -P '' && \
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

RUN mkdir -p ~/hdfs/namenode && \ 
    mkdir -p ~/hdfs/datanode && \
    mkdir $HADOOP_HOME/logs

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
    mv /tmp/hive-site.xml $HIVE_HOME/conf/hive-site.xml

RUN chmod +x ~/start.sh && \
    chmod +x ~/run-wordcount.sh && \
    chmod +x $HADOOP_HOME/sbin/start-dfs.sh && \
    chmod +x $HADOOP_HOME/sbin/start-yarn.sh && \
    chmod 600 ~/.ssh/config

RUN rm ${HIVE_HOME}/lib/guava-19.0.jar && \
    cp ${HADOOP_HOME}/share/hadoop/hdfs/lib/guava-27.0-jre.jar ${HIVE_HOME}/lib/

ADD packages/mysql-connector-java-8.0.23.jar $HIVE_HOME/lib/

# format namenode and init hive schema
RUN /usr/local/hadoop/bin/hdfs namenode -format

CMD service ssh start && tail -F anything

