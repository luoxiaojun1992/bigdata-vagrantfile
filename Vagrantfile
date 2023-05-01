Vagrant.configure("2") do |config|
  config.ssh.insert_key = false

  config.vm.define "bigdata-1" do |bigdata1|
    bigdata1.vm.box = "ubuntu/jammy64"
    bigdata1.vm.hostname = "bigdata1"
    bigdata1.vm.network "private_network", ip: "192.168.33.10"
    bigdata1.vm.network "forwarded_port", guest: 9870, host: 9870
    bigdata1.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end
    bigdata1.vm.provision "shell", inline: <<-SHELL
      if [ ! -f /home/vagrant/init.lock ]; then
        touch /home/vagrant/init.lock

        apt-get install -y openjdk-11-jre-headless
        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /home/vagrant/.bashrc
        echo 'export HADOOP_HOME=/home/vagrant/hadoop' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> /home/vagrant/.bashrc
        echo 'export ZOOKEEPER_HOME=/home/vagrant/zookeeper' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$ZOOKEEPER_HOME/bin' >> /home/vagrant/.bashrc
        echo 'export HBASE_HOME=/home/vagrant/hbase' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$HBASE_HOME/bin' >> /home/vagrant/.bashrc
        echo 'export SPARK_HOME=/home/vagrant/spark' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin' >> /home/vagrant/.bashrc

        sed -i "s#127.0.2.1 bigdata1 bigdata1##g" /etc/hosts
        echo '192.168.33.10 bigdata1' >> /etc/hosts
        echo '192.168.33.11 bigdata2' >> /etc/hosts
        echo '192.168.33.12 bigdata3' >> /etc/hosts

        echo '#! /bin/bash' > /home/vagrant/start-all.sh
        echo 'hdfs namenode -format' >> /home/vagrant/start-all.sh
        echo 'start-all.sh' >> /home/vagrant/start-all.sh
        echo 'zkServer.sh start' >> /home/vagrant/start-all.sh
        chmod +x /home/vagrant/start-all.sh
        chown vagrant /home/vagrant/start-all.sh
      fi

      if [ ! -f /home/vagrant/.ssh/id_rsa ]; then
        curl -L -o /home/vagrant/.ssh/id_rsa https://raw.githubusercontent.com/hashicorp/vagrant/main/keys/vagrant
        chown vagrant /home/vagrant/.ssh/id_rsa
        chmod 600 /home/vagrant/.ssh/id_rsa
      fi

      if [ ! -f /home/vagrant/.ssh/id_rsa.pub ]; then
        curl -L -o /home/vagrant/.ssh/id_rsa.pub https://raw.githubusercontent.com/hashicorp/vagrant/main/keys/vagrant.pub
        cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
        chown vagrant /home/vagrant/.ssh/id_rsa.pub
        chmod 644 /home/vagrant/.ssh/id_rsa.pub
      fi

      if [ ! -f /home/vagrant/hadoop-3.3.5-aarch64.tar.gz ]; then
        wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.5/hadoop-3.3.5-aarch64.tar.gz
        tar -zxvf hadoop-3.3.5-aarch64.tar.gz
        mv hadoop-3.3.5 hadoop
        chown vagrant -R hadoop
        
        echo 'bigdata1' > /home/vagrant/hadoop/etc/hadoop/workers
        echo 'bigdata2' >> /home/vagrant/hadoop/etc/hadoop/workers
        echo 'bigdata3' >> /home/vagrant/hadoop/etc/hadoop/workers

        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/hadoop/etc/hadoop/hadoop-env.sh
        echo 'export HADOOP_HOME=/home/vagrant/hadoop' >> /home/vagrant/hadoop/etc/hadoop/hadoop-env.sh

        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>fs.defaultFS<\/name>\\n    <value>hdfs:\/\/bigdata1:9000<\/value>\\n  <\/property>\\n  <property>\\n    <name>hadoop.tmp.dir<\/name>\\n    <value>\/home\/vagrant\/hadoop\/data\/tmp<\/value>\\n  <\/property>\\n  <property>\\n    <name>hadoop.http.staticuser.user<\/name>\\n    <value>vagrant<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/core-site.xml
        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>dfs.replication<\/name>\\n    <value>3<\/value>\\n  <\/property>\\n  <property>\\n    <name>dfs.namenode.secondary.http-address<\/name>\\n    <value>bigdata2:50090<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/hdfs-site.xml
        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>yarn.nodemanager.aux-services<\/name>\\n    <value>mapreduce_shuffle<\/value>\\n  <\/property>\\n  <property>\\n    <name>yarn.resourcemanager.hostname<\/name>\\n    <value>bigdata2<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/yarn-site.xml
        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>mapreduce.framework.name<\/name>\\n    <value>yarn<\/value>\\n  <\/property>\\n  <property>\\n    <name>mapreduce.map.env<\/name>\\n    <value>HADOOP_MAPRED_HOME=\/home\/vagrant\/hadoop<\/value>\\n  <\/property>\\n  <property>\\n    <name>mapreduce.reduce.env<\/name>\\n    <value>HADOOP_MAPRED_HOME=\/home\/vagrant\/hadoop<\/value>\\n  <\/property>\\n  <property>\\n    <name>mapreduce.application.classpath<\/name>\\n    <value>\/home\/vagrant\/hadoop\/share\/hadoop\/mapreduce\/*:\/home\/vagrant\/hadoop\/share\/hadoop\/mapreduce\/lib\/*<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/mapred-site.xml
      
        touch /home/vagrant/bigdata1_hadoop_ready
      fi

      if [ ! -f /home/vagrant/apache-zookeeper-3.8.1-bin.tar.gz ]; then
        wget https://dlcdn.apache.org/zookeeper/zookeeper-3.8.1/apache-zookeeper-3.8.1-bin.tar.gz
        tar -zxvf apache-zookeeper-3.8.1-bin.tar.gz
        mv apache-zookeeper-3.8.1-bin zookeeper

        cp /home/vagrant/zookeeper/conf/zoo_sample.cfg /home/vagrant/zookeeper/conf/zoo.cfg
        sed -i "s#dataDir=\/tmp\/zookeeper#dataDir=\/home\/vagrant\/zookeeper\/data#g" /home/vagrant/zookeeper/conf/zoo.cfg
        echo 'server.1=0.0.0.0:2888:3888' >> /home/vagrant/zookeeper/conf/zoo.cfg
        echo 'server.2=bigdata2:2888:3888' >> /home/vagrant/zookeeper/conf/zoo.cfg
        echo 'server.3=bigdata3:2888:3888' >> /home/vagrant/zookeeper/conf/zoo.cfg
        mkdir -p /home/vagrant/zookeeper/data
        echo '1' > /home/vagrant/zookeeper/data/myid

        chown vagrant -R /home/vagrant/zookeeper

        touch /home/vagrant/bigdata1_zookeeper_ready
      fi

      if [ ! -f /home/vagrant/hbase-2.4.17-bin.tar.gz ]; then
        wget https://dlcdn.apache.org/hbase/2.4.17/hbase-2.4.17-bin.tar.gz
        tar -zxvf hbase-2.4.17-bin.tar.gz
        mv hbase-2.4.17 hbase

        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HADOOP_HOME=/home/vagrant/hadoop' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_HOME=/home/vagrant/hbase' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_CLASSPATH=/home/vagrant/hadoop/etc/hadoop' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_PID_DIR=/home/vagrant/hbase/pids' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_MANAGES_ZK=false' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_DISABLE_HADOOP_CLASSPATH_LOOKUP="true"' >> /home/vagrant/hbase/conf/hbase-env.sh

        echo 'bigdata1' > /home/vagrant/hbase/conf/regionservers
        echo 'bigdata2' >> /home/vagrant/hbase/conf/regionservers
        echo 'bigdata3' >> /home/vagrant/hbase/conf/regionservers

        mv /home/vagrant/hbase/conf/hbase-site.xml /home/vagrant/hbase/conf/hbase-site.xml.template
        echo '<?xml version="1.0"?>' > /home/vagrant/hbase/conf/hbase-site.xml
        echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '<configuration>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.cluster.distributed</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>true</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.tmp.dir</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>/home/vagrant/hbase/tmp</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.unsafe.stream.capability.enforce</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>false</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.rootdir</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>hdfs://bigdata1:9000/hbase</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.zookeeper.quorum</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>bigdata1,bigdata2,bigdata3</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.zookeeper.property.clientPort</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>2181</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '</configuration>' >> /home/vagrant/hbase/conf/hbase-site.xml

        chown vagrant -R /home/vagrant/hbase

        touch /home/vagrant/bigdata1_hbase_ready
      fi

      if [ ! -f /home/vagrant/spark-3.4.0-bin-without-hadoop.tgz ]; then
        wget https://dlcdn.apache.org/spark/spark-3.4.0/spark-3.4.0-bin-without-hadoop.tgz
        tar -zxvf spark-3.4.0-bin-without-hadoop.tgz
        mv spark-3.4.0-bin-without-hadoop spark

        mv /home/vagrant/spark/sbin/start-all.sh /home/vagrant/spark/sbin/start-spark-all.sh
        mv /home/vagrant/spark/sbin/stop-all.sh /home/vagrant/spark/sbin/stop-spark-all.sh

        cp /home/vagrant/spark/conf/spark-env.sh.template /home/vagrant/spark/conf/spark-env.sh
        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export HADOOP_CONF_DIR=/home/vagrant/hadoop/etc/hadoop/' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_MASTER_HOST=bigdata1' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_MASTER_PORT=27077' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_MASTER_WEBUI_PORT=28080' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_WORKER_CORES=2' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_WORKER_MEMORY=2g' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_DIST_CLASSPATH=$(/home/vagrant/hadoop/bin/hadoop classpath)' >> /home/vagrant/spark/conf/spark-env.sh

        cp /home/vagrant/spark/conf/workers.template /home/vagrant/spark/conf/workers
        echo 'bigdata2' > /home/vagrant/spark/conf/workers
        echo 'bigdata3' >> /home/vagrant/spark/conf/workers

        chown vagrant -R /home/vagrant/spark

        touch /home/vagrant/bigdata1_spark_ready
      fi
    SHELL
  end

  config.vm.define "bigdata-2" do |bigdata2|
    bigdata2.vm.box = "ubuntu/jammy64"
    bigdata2.vm.hostname = "bigdata2"
    bigdata2.vm.network "private_network", ip: "192.168.33.11"
    bigdata2.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end
    bigdata2.vm.provision "shell", inline: <<-SHELL
      if [ ! -f /home/vagrant/init.lock ]; then
        touch /home/vagrant/init.lock

        apt-get install -y openjdk-11-jre-headless
        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /home/vagrant/.bashrc
        echo 'export HADOOP_HOME=/home/vagrant/hadoop' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> /home/vagrant/.bashrc
        echo 'export ZOOKEEPER_HOME=/home/vagrant/zookeeper' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$ZOOKEEPER_HOME/bin' >> /home/vagrant/.bashrc
        echo 'export HBASE_HOME=/home/vagrant/hbase' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$HBASE_HOME/bin' >> /home/vagrant/.bashrc
        echo 'export SPARK_HOME=/home/vagrant/spark' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin' >> /home/vagrant/.bashrc

        sed -i "s#127.0.2.1 bigdata2 bigdata2##g" /etc/hosts
        echo '192.168.33.10 bigdata1' >> /etc/hosts
        echo '192.168.33.11 bigdata2' >> /etc/hosts
        echo '192.168.33.12 bigdata3' >> /etc/hosts

        echo '#! /bin/bash' > /home/vagrant/start-all.sh
        echo 'zkServer.sh start' >> /home/vagrant/start-all.sh
        chmod +x /home/vagrant/start-all.sh
        chown vagrant /home/vagrant/start-all.sh
      fi

      if [ ! -f /home/vagrant/.ssh/id_rsa ]; then
        curl -L -o /home/vagrant/.ssh/id_rsa https://raw.githubusercontent.com/hashicorp/vagrant/main/keys/vagrant
        chown vagrant /home/vagrant/.ssh/id_rsa
        chmod 600 /home/vagrant/.ssh/id_rsa
      fi

      if [ ! -f /home/vagrant/.ssh/id_rsa.pub ]; then
        curl -L -o /home/vagrant/.ssh/id_rsa.pub https://raw.githubusercontent.com/hashicorp/vagrant/main/keys/vagrant.pub
        cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
        chown vagrant /home/vagrant/.ssh/id_rsa.pub
        chmod 644 /home/vagrant/.ssh/id_rsa.pub
      fi

      if [ ! -f /home/vagrant/hadoop-3.3.5-aarch64.tar.gz ]; then
        wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.5/hadoop-3.3.5-aarch64.tar.gz
        tar -zxvf hadoop-3.3.5-aarch64.tar.gz
        mv hadoop-3.3.5 hadoop
        chown vagrant -R hadoop
        
        echo 'bigdata1' > /home/vagrant/hadoop/etc/hadoop/workers
        echo 'bigdata2' >> /home/vagrant/hadoop/etc/hadoop/workers
        echo 'bigdata3' >> /home/vagrant/hadoop/etc/hadoop/workers

        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/hadoop/etc/hadoop/hadoop-env.sh
        echo 'export HADOOP_HOME=/home/vagrant/hadoop' >> /home/vagrant/hadoop/etc/hadoop/hadoop-env.sh

        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>fs.defaultFS<\/name>\\n    <value>hdfs:\/\/bigdata1:9000<\/value>\\n  <\/property>\\n  <property>\\n    <name>hadoop.tmp.dir<\/name>\\n    <value>\/home\/vagrant\/hadoop\/data\/tmp<\/value>\\n  <\/property>\\n  <property>\\n    <name>hadoop.http.staticuser.user<\/name>\\n    <value>vagrant<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/core-site.xml
        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>dfs.replication<\/name>\\n    <value>3<\/value>\\n  <\/property>\\n  <property>\\n    <name>dfs.namenode.secondary.http-address<\/name>\\n    <value>bigdata2:50090<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/hdfs-site.xml
        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>yarn.nodemanager.aux-services<\/name>\\n    <value>mapreduce_shuffle<\/value>\\n  <\/property>\\n  <property>\\n    <name>yarn.resourcemanager.hostname<\/name>\\n    <value>bigdata2<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/yarn-site.xml
        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>mapreduce.framework.name<\/name>\\n    <value>yarn<\/value>\\n  <\/property>\\n  <property>\\n    <name>mapreduce.map.env<\/name>\\n    <value>HADOOP_MAPRED_HOME=\/home\/vagrant\/hadoop<\/value>\\n  <\/property>\\n  <property>\\n    <name>mapreduce.reduce.env<\/name>\\n    <value>HADOOP_MAPRED_HOME=\/home\/vagrant\/hadoop<\/value>\\n  <\/property>\\n  <property>\\n    <name>mapreduce.application.classpath<\/name>\\n    <value>\/home\/vagrant\/hadoop\/share\/hadoop\/mapreduce\/*:\/home\/vagrant\/hadoop\/share\/hadoop\/mapreduce\/lib\/*<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/mapred-site.xml
      
        touch /home/vagrant/bigdata2_hadoop_ready
      fi

      if [ ! -f /home/vagrant/apache-zookeeper-3.8.1-bin.tar.gz ]; then
        wget https://dlcdn.apache.org/zookeeper/zookeeper-3.8.1/apache-zookeeper-3.8.1-bin.tar.gz
        tar -zxvf apache-zookeeper-3.8.1-bin.tar.gz
        mv apache-zookeeper-3.8.1-bin zookeeper

        cp /home/vagrant/zookeeper/conf/zoo_sample.cfg /home/vagrant/zookeeper/conf/zoo.cfg
        sed -i "s#dataDir=\/tmp\/zookeeper#dataDir=\/home\/vagrant\/zookeeper\/data#g" /home/vagrant/zookeeper/conf/zoo.cfg
        echo 'server.1=bigdata1:2888:3888' >> /home/vagrant/zookeeper/conf/zoo.cfg
        echo 'server.2=0.0.0.0:2888:3888' >> /home/vagrant/zookeeper/conf/zoo.cfg
        echo 'server.3=bigdata3:2888:3888' >> /home/vagrant/zookeeper/conf/zoo.cfg
        mkdir -p /home/vagrant/zookeeper/data
        echo '2' > /home/vagrant/zookeeper/data/myid

        chown vagrant -R /home/vagrant/zookeeper

        touch /home/vagrant/bigdata2_zookeeper_ready
      fi

      if [ ! -f /home/vagrant/hbase-2.4.17-bin.tar.gz ]; then
        wget https://dlcdn.apache.org/hbase/2.4.17/hbase-2.4.17-bin.tar.gz
        tar -zxvf hbase-2.4.17-bin.tar.gz
        mv hbase-2.4.17 hbase

        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HADOOP_HOME=/home/vagrant/hadoop' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_HOME=/home/vagrant/hbase' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_CLASSPATH=/home/vagrant/hadoop/etc/hadoop' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_PID_DIR=/home/vagrant/hbase/pids' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_MANAGES_ZK=false' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_DISABLE_HADOOP_CLASSPATH_LOOKUP="true"' >> /home/vagrant/hbase/conf/hbase-env.sh

        echo 'bigdata1' > /home/vagrant/hbase/conf/regionservers
        echo 'bigdata2' >> /home/vagrant/hbase/conf/regionservers
        echo 'bigdata3' >> /home/vagrant/hbase/conf/regionservers

        mv /home/vagrant/hbase/conf/hbase-site.xml /home/vagrant/hbase/conf/hbase-site.xml.template
        echo '<?xml version="1.0"?>' > /home/vagrant/hbase/conf/hbase-site.xml
        echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '<configuration>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.cluster.distributed</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>true</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.tmp.dir</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>/home/vagrant/hbase/tmp</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.unsafe.stream.capability.enforce</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>false</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.rootdir</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>hdfs://bigdata1:9000/hbase</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.zookeeper.quorum</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>bigdata1,bigdata2,bigdata3</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.zookeeper.property.clientPort</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>2181</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '</configuration>' >> /home/vagrant/hbase/conf/hbase-site.xml

        chown vagrant -R /home/vagrant/hbase

        touch /home/vagrant/bigdata2_hbase_ready
      fi

      if [ ! -f /home/vagrant/spark-3.4.0-bin-without-hadoop.tgz ]; then
        wget https://dlcdn.apache.org/spark/spark-3.4.0/spark-3.4.0-bin-without-hadoop.tgz
        tar -zxvf spark-3.4.0-bin-without-hadoop.tgz
        mv spark-3.4.0-bin-without-hadoop spark

        mv /home/vagrant/spark/sbin/start-all.sh /home/vagrant/spark/sbin/start-spark-all.sh
        mv /home/vagrant/spark/sbin/stop-all.sh /home/vagrant/spark/sbin/stop-spark-all.sh

        cp /home/vagrant/spark/conf/spark-env.sh.template /home/vagrant/spark/conf/spark-env.sh
        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export HADOOP_CONF_DIR=/home/vagrant/hadoop/etc/hadoop/' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_MASTER_HOST=bigdata1' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_MASTER_PORT=27077' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_MASTER_WEBUI_PORT=28080' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_WORKER_CORES=2' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_WORKER_MEMORY=2g' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_DIST_CLASSPATH=$(/home/vagrant/hadoop/bin/hadoop classpath)' >> /home/vagrant/spark/conf/spark-env.sh

        cp /home/vagrant/spark/conf/workers.template /home/vagrant/spark/conf/workers
        echo 'bigdata2' > /home/vagrant/spark/conf/workers
        echo 'bigdata3' >> /home/vagrant/spark/conf/workers

        chown vagrant -R /home/vagrant/spark

        touch /home/vagrant/bigdata2_spark_ready
      fi
    SHELL
  end

  config.vm.define "bigdata-3" do |bigdata3|
    bigdata3.vm.box = "ubuntu/jammy64"
    bigdata3.vm.hostname = "bigdata3"
    bigdata3.vm.network "private_network", ip: "192.168.33.12"
    bigdata3.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end
    bigdata3.vm.provision "shell", inline: <<-SHELL
      if [ ! -f /home/vagrant/init.lock ]; then
        touch /home/vagrant/init.lock

        apt-get install -y openjdk-11-jre-headless
        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /home/vagrant/.bashrc
        echo 'export HADOOP_HOME=/home/vagrant/hadoop' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> /home/vagrant/.bashrc
        echo 'export ZOOKEEPER_HOME=/home/vagrant/zookeeper' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$ZOOKEEPER_HOME/bin' >> /home/vagrant/.bashrc
        echo 'export HBASE_HOME=/home/vagrant/hbase' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$HBASE_HOME/bin' >> /home/vagrant/.bashrc
        echo 'export SPARK_HOME=/home/vagrant/spark' >> /home/vagrant/.bashrc
        echo 'export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin' >> /home/vagrant/.bashrc

        sed -i "s#127.0.2.1 bigdata3 bigdata3##g" /etc/hosts
        echo '192.168.33.10 bigdata1' >> /etc/hosts
        echo '192.168.33.11 bigdata2' >> /etc/hosts
        echo '192.168.33.12 bigdata3' >> /etc/hosts

        echo '#! /bin/bash' > /home/vagrant/start-all.sh
        echo 'zkServer.sh start' >> /home/vagrant/start-all.sh
        chmod +x /home/vagrant/start-all.sh
        chown vagrant /home/vagrant/start-all.sh
      fi

      if [ ! -f /home/vagrant/.ssh/id_rsa ]; then
        curl -L -o /home/vagrant/.ssh/id_rsa https://raw.githubusercontent.com/hashicorp/vagrant/main/keys/vagrant
        chown vagrant /home/vagrant/.ssh/id_rsa
        chmod 600 /home/vagrant/.ssh/id_rsa
      fi

      if [ ! -f /home/vagrant/.ssh/id_rsa.pub ]; then
        curl -L -o /home/vagrant/.ssh/id_rsa.pub https://raw.githubusercontent.com/hashicorp/vagrant/main/keys/vagrant.pub
        cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
        chown vagrant /home/vagrant/.ssh/id_rsa.pub
        chmod 644 /home/vagrant/.ssh/id_rsa.pub
      fi

      if [ ! -f /home/vagrant/hadoop-3.3.5-aarch64.tar.gz ]; then
        wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.5/hadoop-3.3.5-aarch64.tar.gz
        tar -zxvf hadoop-3.3.5-aarch64.tar.gz
        mv hadoop-3.3.5 hadoop
        chown vagrant -R hadoop
        
        echo 'bigdata1' > /home/vagrant/hadoop/etc/hadoop/workers
        echo 'bigdata2' >> /home/vagrant/hadoop/etc/hadoop/workers
        echo 'bigdata3' >> /home/vagrant/hadoop/etc/hadoop/workers

        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/hadoop/etc/hadoop/hadoop-env.sh
        echo 'export HADOOP_HOME=/home/vagrant/hadoop' >> /home/vagrant/hadoop/etc/hadoop/hadoop-env.sh

        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>fs.defaultFS<\/name>\\n    <value>hdfs:\/\/bigdata1:9000<\/value>\\n  <\/property>\\n  <property>\\n    <name>hadoop.tmp.dir<\/name>\\n    <value>\/home\/vagrant\/hadoop\/data\/tmp<\/value>\\n  <\/property>\\n  <property>\\n    <name>hadoop.http.staticuser.user<\/name>\\n    <value>vagrant<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/core-site.xml
        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>dfs.replication<\/name>\\n    <value>3<\/value>\\n  <\/property>\\n  <property>\\n    <name>dfs.namenode.secondary.http-address<\/name>\\n    <value>bigdata2:50090<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/hdfs-site.xml
        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>yarn.nodemanager.aux-services<\/name>\\n    <value>mapreduce_shuffle<\/value>\\n  <\/property>\\n  <property>\\n    <name>yarn.resourcemanager.hostname<\/name>\\n    <value>bigdata2<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/yarn-site.xml
        sed -i "s#<configuration>#<configuration>\\n  <property>\\n    <name>mapreduce.framework.name<\/name>\\n    <value>yarn<\/value>\\n  <\/property>\\n  <property>\\n    <name>mapreduce.map.env<\/name>\\n    <value>HADOOP_MAPRED_HOME=\/home\/vagrant\/hadoop<\/value>\\n  <\/property>\\n  <property>\\n    <name>mapreduce.reduce.env<\/name>\\n    <value>HADOOP_MAPRED_HOME=\/home\/vagrant\/hadoop<\/value>\\n  <\/property>\\n  <property>\\n    <name>mapreduce.application.classpath<\/name>\\n    <value>\/home\/vagrant\/hadoop\/share\/hadoop\/mapreduce\/*:\/home\/vagrant\/hadoop\/share\/hadoop\/mapreduce\/lib\/*<\/value>\\n  <\/property>#g" /home/vagrant/hadoop/etc/hadoop/mapred-site.xml

        touch /home/vagrant/bigdata3_hadoop_ready
      fi

      if [ ! -f /home/vagrant/apache-zookeeper-3.8.1-bin.tar.gz ]; then
        wget https://dlcdn.apache.org/zookeeper/zookeeper-3.8.1/apache-zookeeper-3.8.1-bin.tar.gz
        tar -zxvf apache-zookeeper-3.8.1-bin.tar.gz
        mv apache-zookeeper-3.8.1-bin zookeeper

        cp /home/vagrant/zookeeper/conf/zoo_sample.cfg /home/vagrant/zookeeper/conf/zoo.cfg
        sed -i "s#dataDir=\/tmp\/zookeeper#dataDir=\/home\/vagrant\/zookeeper\/data#g" /home/vagrant/zookeeper/conf/zoo.cfg
        echo 'server.1=bigdata1:2888:3888' >> /home/vagrant/zookeeper/conf/zoo.cfg
        echo 'server.2=bigdata2:2888:3888' >> /home/vagrant/zookeeper/conf/zoo.cfg
        echo 'server.3=0.0.0.0:2888:3888' >> /home/vagrant/zookeeper/conf/zoo.cfg
        mkdir -p /home/vagrant/zookeeper/data
        echo '3' > /home/vagrant/zookeeper/data/myid

        chown vagrant -R /home/vagrant/zookeeper

        touch /home/vagrant/bigdata3_zookeeper_ready
      fi

      if [ ! -f /home/vagrant/hbase-2.4.17-bin.tar.gz ]; then
        wget https://dlcdn.apache.org/hbase/2.4.17/hbase-2.4.17-bin.tar.gz
        tar -zxvf hbase-2.4.17-bin.tar.gz
        mv hbase-2.4.17 hbase

        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HADOOP_HOME=/home/vagrant/hadoop' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_HOME=/home/vagrant/hbase' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_CLASSPATH=/home/vagrant/hadoop/etc/hadoop' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_PID_DIR=/home/vagrant/hbase/pids' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_MANAGES_ZK=false' >> /home/vagrant/hbase/conf/hbase-env.sh
        echo 'export HBASE_DISABLE_HADOOP_CLASSPATH_LOOKUP="true"' >> /home/vagrant/hbase/conf/hbase-env.sh

        echo 'bigdata1' > /home/vagrant/hbase/conf/regionservers
        echo 'bigdata2' >> /home/vagrant/hbase/conf/regionservers
        echo 'bigdata3' >> /home/vagrant/hbase/conf/regionservers

        mv /home/vagrant/hbase/conf/hbase-site.xml /home/vagrant/hbase/conf/hbase-site.xml.template
        echo '<?xml version="1.0"?>' > /home/vagrant/hbase/conf/hbase-site.xml
        echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '<configuration>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.cluster.distributed</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>true</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.tmp.dir</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>/home/vagrant/hbase/tmp</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.unsafe.stream.capability.enforce</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>false</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.rootdir</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>hdfs://bigdata1:9000/hbase</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.zookeeper.quorum</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>bigdata1,bigdata2,bigdata3</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  <property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <name>hbase.zookeeper.property.clientPort</name>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '    <value>2181</value>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '  </property>' >> /home/vagrant/hbase/conf/hbase-site.xml
        echo '</configuration>' >> /home/vagrant/hbase/conf/hbase-site.xml

        chown vagrant -R /home/vagrant/hbase

        touch /home/vagrant/bigdata3_hbase_ready
      fi

      if [ ! -f /home/vagrant/spark-3.4.0-bin-without-hadoop.tgz ]; then
        wget https://dlcdn.apache.org/spark/spark-3.4.0/spark-3.4.0-bin-without-hadoop.tgz
        tar -zxvf spark-3.4.0-bin-without-hadoop.tgz
        mv spark-3.4.0-bin-without-hadoop spark

        mv /home/vagrant/spark/sbin/start-all.sh /home/vagrant/spark/sbin/start-spark-all.sh
        mv /home/vagrant/spark/sbin/stop-all.sh /home/vagrant/spark/sbin/stop-spark-all.sh

        cp /home/vagrant/spark/conf/spark-env.sh.template /home/vagrant/spark/conf/spark-env.sh
        echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export HADOOP_CONF_DIR=/home/vagrant/hadoop/etc/hadoop/' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_MASTER_HOST=bigdata1' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_MASTER_PORT=27077' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_MASTER_WEBUI_PORT=28080' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_WORKER_CORES=2' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_WORKER_MEMORY=2g' >> /home/vagrant/spark/conf/spark-env.sh
        echo 'export SPARK_DIST_CLASSPATH=$(/home/vagrant/hadoop/bin/hadoop classpath)' >> /home/vagrant/spark/conf/spark-env.sh

        cp /home/vagrant/spark/conf/workers.template /home/vagrant/spark/conf/workers
        echo 'bigdata2' > /home/vagrant/spark/conf/workers
        echo 'bigdata3' >> /home/vagrant/spark/conf/workers

        chown vagrant -R /home/vagrant/spark

        touch /home/vagrant/bigdata3_spark_ready
      fi
    SHELL
  end
end
