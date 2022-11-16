FROM ubuntu
LABEL maintainer="Starry"
RUN apt update && apt install -y openssh-server openssh-client vim openjdk-8-jdk
RUN apt update && apt install -y scala git

# SSH
RUN ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
RUN cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
RUN sed -ri 's/#   StrictHostKeyChecking ask/    StrictHostKeyChecking accept-new/g' /etc/ssh/ssh_config
RUN chmod 0600 ~/.ssh/authorized_keys

RUN echo "root:123" | chpasswd
RUN echo "root   ALL=(ALL)       ALL" >> /etc/sudoers

ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64
ENV PATH $JAVA_HOME/bin:$PATH

# Hadoop
RUN wget https://dlcdn.apache.org/hadoop/core/hadoop-3.3.1/hadoop-3.3.1.tar.gz
RUN tar -xzf hadoop-3.3.1.tar.gz
RUN mv hadoop-3.3.1 usr/local/hadoop
ENV HADOOP_HOME /usr/local/hadoop
ENV HADOOP_CONF_DIR $HADOOP_HOME/etc/hadoop
ENV PATH $HADOOP_HOME/sbin:$PATH

RUN mkdir /home/hadoop
RUN mkdir /home/hadoop/tmp /home/hadoop/hdfs_name /home/hadoop/hdfs_data

ADD config/core-site.xml $HADOOP_HOME/etc/hadoop/core-site.xml
ADD config/hdfs-site.xml $HADOOP_HOME/etc/hadoop/hdfs-site.xml
ADD config/mapred-site.xml $HADOOP_HOME/etc/hadoop/mapred-site.xml
ADD config/yarn-site.xml $HADOOP_HOME/etc/hadoop/yarn-site.xml
ADD config/slaves $HADOOP_HOME/etc/hadoop/slaves
ADD config/start-dfs.sh $HADOOP_HOME/sbin
ADD config/stop-dfs.sh $HADOOP_HOME/sbin
ADD config/start-yarn.sh $HADOOP_HOME/sbin
ADD config/stop-yarn.sh $HADOOP_HOME/sbin

RUN echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

ENV PATH $HADOOP_HOME/bin:$PATH

# Mysql 
RUN wget http://ftp.ntu.edu.tw/MySQL/Downloads/Connector-J/mysql-connector-java-8.0.26.tar.gz
RUN tar -xvf mysql-connector-java-8.0.26.tar.gz

# Hive
RUN wget https://mirror.downloadvn.com/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
RUN tar -xzf apache-hive-3.1.2-bin.tar.gz
RUN mv apache-hive-3.1.2-bin /usr/local/hive
ENV HIVE_HOME /usr/local/hive
ENV PATH $HIVE_HOME/bin:$PATH
RUN echo "HADOOP_HOME=/usr/local/hadoop" >> $HIVE_HOME/bin/hive-config.sh
RUN cp mysql-connector-java-8.0.26/mysql-connector-java-8.0.26.jar $HIVE_HOME/lib/
RUN sh -c '/usr/local/hive/bin/schematool -dbType mysql -initSchema'
ADD config/hive-site.xml $HIVE_HOME/conf/

# Spark
RUN wget https://downloads.apache.org/spark/spark-3.0.3/spark-3.0.3-bin-hadoop3.2.tgz
RUN tar xvf spark-3.0.3-bin-hadoop3.2.tgz
RUN mv spark-3.0.3-bin-hadoop3.2 /usr/local/spark
ENV SPARK_HOME /usr/local/spark
ENV PATH $SPARK_HOME/sbin:$PATH
ENV PATH $SPARK_HOME/bin:$PATH
ADD config/hive-site.xml $SPARK_HOME/conf/
RUN cp mysql-connector-java-8.0.26/mysql-connector-java-8.0.26.jar $SPARK_HOME/jars/

ADD config/slaves $SPARK_HOME/conf/slaves
RUN echo "export SPARK_MASTER_HOST='172.12.0.2'" >> $SPARK_HOME/conf/spark-env.sh
RUN echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> $SPARK_HOME/conf/spark-env.sh
RUN echo "export SPARK_CLASSPATH=$SPARK_HOME/jars/mysql-connector-java-8.0.26.jar" >> $SPARK_HOME/conf/spark-env.sh
RUN echo "export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop " >> $SPARK_HOME/conf/spark-env.sh
RUN echo "export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop" >> $SPARK_HOME/conf/spark-env.sh

# Zeppelin
ENV ZEPPELIN_VERSION=0.10.0
ENV SPARK_WORKER_WEBUI_PORT 8082
ENV SPARK_WORKER_LOG /spark/logs
ENV SPARK_MASTER "spark://spark-master:7077"
ENV SPARK_APP_NAME zeppelin
RUN wget http://dlcdn.apache.org/zeppelin/zeppelin-${ZEPPELIN_VERSION}/zeppelin-${ZEPPELIN_VERSION}-bin-netinst.tgz -O /tmp/zeppelin.tgz \
  && mkdir /usr/local/zeppelin \
  && tar zxf /tmp/zeppelin.tgz -C /usr/loc al/zeppelin --strip-components 1
COPY config/zeppelin-site.xml /zeppelin/conf/zeppelin-site.xml
ENV PATH=/zeppelin/bin:$PATH

ARG FORMAT_NAMENODE_COMMAND
RUN $FORMAT_NAMENODE_COMMAND
RUN mkdir -p /run/sshd
RUN /usr/sbin/sshd
EXPOSE 22
