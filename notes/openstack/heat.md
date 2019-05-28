# heat相关

```
read -p "how many groups do you want:" groups
read -p "private_net_name:" private_net_name

for((i=1;i<=$groups;i++));do
heat stack-create -f /root/bigdata.yaml -P "private_net_name="$private_net_name$i"_private_net" hadoop_$i
done
```

```
heat_template_version: 2013-05-23

description: >
  HOT template to create a new neutron network plus a router to the public
  network, and for deploying two servers into the new network. The template also
  assigns floating IP addresses to each server so they are routable from the
  public network.

parameters:
  myimage:
    type: string
    description: testsssssssssssssssssssssssss
    constraints:
      - custom_constraint: glance.image
    default: c1026774-a7ed-4dd3-8b1e-0f67b87b42af

  myname:
    type: string
    description: myname
    default: ttttt

  key_name:
    type: string
    description: Name of keypair to assign to servers
    constraints:
      - custom_constraint: nova.keypair
    default: mykey

  flavor:
    type: string
    description: Flavor to use for servers
    constraints:
      - custom_constraint: nova.flavor
    default: 7377696f-dd6b-41bd-85ec-2043dbebcc5e

  networks:
    type: string
    description: Name of an existing to use for the server
    constraints:
      - custom_constraint: neutron.network


resources:
  servertest:
    type: OS::Nova::Server
    properties:
      name: { get_param: myname }
      image: { get_param: myimage }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks: [{ 'network':{ get_param: networks}}]
```
```
heat_template_version: 2013-05-23

description: >
  HOT template to create a new neutron network plus a router to the public
  network, and for deploying two servers into the new network. The template also
  assigns floating IP addresses to each server so they are routable from the
  public network.
parameters:
  key_name:
    type: string
    description: Name of keypair to assign to servers
    constraints:
      - custom_constraint: nova.keypair
    default: mykey
  image:
    type: string
    description: Name of image to use for servers
    constraints:
      - custom_constraint: glance.image
    default: 16fa0162-35d7-4e92-8591-4f60f88c57f3
  image_desktop:
    type: string
    description: Name of image to use for servers
    constraints:
      - custom_constraint: glance.image
    default: 0b0147f5-34d2-47c3-b20b-82513caf6f83 
  flavor:
    type: string
    description: Flavor to use for servers
    constraints:
      - custom_constraint: nova.flavor
    default: 335eb246-f1c9-426f-a4da-36c5e71741a9
  flavor_desktop:
    type: string
    description: Flavor to use for desktops
    constraints:
      - custom_constraint: nova.flavor
    default: 335eb246-f1c9-426f-a4da-36c5e71741a9
  login_pass:
    type: string
    description: Login password
    hidden: false
    default: 123456
  public_net:
    type: string
    description: >
      ID or name of public network for which floating IP addresses will be allocated
    constraints:
      - custom_constraint: neutron.network
    default: 1f6a08a2-421e-4449-a768-095426d829e8
  private_net_name:
    type: string
    description: Name of private network to be created
  private_net_cidr:
    type: string
    description: Private network address (CIDR notation)
    default: 192.168.1.0/24
  private_net_gateway:
    type: string
    description: Private network gateway address
    default: 192.168.1.254
  private_net_pool_begin:
    type: string
    description: Start of private network IP address allocation pool
    default: 192.168.1.1
  private_net_pool_end:
    type: string
    description: End of private network IP address allocation pool
    default: 192.168.1.253
  dns_nameservers:
    type: string
    description: A specified set of DNS name servers to be used
    default: 192.168.90.127

resources:
  private_net:
    type: OS::Neutron::Net
    properties:
      name: { get_param: private_net_name }

  private_subnet:
    type: OS::Neutron::Subnet
    properties:
      network_id: { get_resource: private_net }
      cidr: { get_param: private_net_cidr }
      gateway_ip: { get_param: private_net_gateway }
      dns_nameservers: [{ get_param: dns_nameservers}]
      allocation_pools:
        - start: { get_param: private_net_pool_begin }
          end: { get_param: private_net_pool_end }

  router:
    type: OS::Neutron::Router
    properties:
      external_gateway_info:
        network: { get_param: public_net }

  router_interface:
    type: OS::Neutron::RouterInterface
    properties:
      router_id: { get_resource: router }
      subnet_id: { get_resource: private_subnet }

  server1_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: private_net }
      fixed_ips: [{ 'ip_address': 192.168.1.2}]

  server1_floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: { get_param: public_net }
      port_id: { get_resource: server1_port }

  server1:
    type: OS::Nova::Server
    properties:
      name: Master
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: server1_port }
      user_data:
        str_replace:
          template: |
            #!/bin/bash

            #change login password
    
            /usr/bin/expect << EOF
            spawn sudo passwd root
            set timeout -1
            expect {
                    "Enter new UNIX password:" { 
                            send "login_pass\n"; 
                            exp_continue 
                    }
                    "Retype new UNIX password:" { 
                            send "login_pass\n"
                    }
            }
            expect eof
            EOF
    
            #pull packages
            scp_ip=192.168.65.241
            scp_passwd=handgecloud
    
            /usr/bin/expect << EOF
            spawn scp -r root@$scp_ip:/mnt/ramdisk/opt.tgz /
            expect {
                    "*(yes/no)?" { 
                            send "yes\n"; 
                            exp_continue 
                    }
                    "*password:" { 
                            send "$scp_passwd\n"
                    }
            }
            set timeout -1
            expect eof
            EOF
    
            cd /
            pigz -d /opt.tgz
            tar -xf /opt.tar
            rm /opt.tar
    
            #vim hosts
            echo "192.168.1.2 Master master" >> /etc/hosts
            echo "192.168.1.3 Slave1 slave1" >> /etc/hosts
            echo "192.168.1.4 Slave2 slave2" >> /etc/hosts
            sed -i "s/127.0.1.1/ /g" /etc/hosts
            sed -i "s/ubuntu/ /g" /etc/hosts
    
            #ENV
            echo "export JAVA_HOME=/opt/jdk1.8.0_111" >> ~/.bashrc
            echo "export HADOOP_HOME=/opt/hadoop-2.7.3" >> ~/.bashrc
            echo "export HBASE_HOME=/opt/hbase-1.2.3" >> ~/.bashrc
            echo "export ZOOKEEPER_HOME=/opt/zookeeper-3.4.9" >> ~/.bashrc
            echo 'export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$JAVA_HOME/bin:$HBASE_HOME/bin:$HBASE_HOME/sbin:$ZOOKEEPER_HOME/bin:$PATH' >> ~/.bashrc
            source ~/.bashrc
            sed -i 's/JAVA_HOME=${JAVA_HOME}/JAVA_HOME=\/opt\/jdk1.8.0_111/g' /opt/hadoop-2.7.3/etc/hadoop/hadoop-env.sh
    
            ##########################HADOOP####################
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/slaves
            echo "Slave1" > /opt/hadoop-2.7.3/etc/hadoop/slaves
            echo "Slave2" >> /opt/hadoop-2.7.3/etc/hadoop/slaves
    
            touch /opt/hadoop-2.7.3/etc/hadoop/master
            echo "Master" > /opt/hadoop-2.7.3/etc/hadoop/master
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '20i <property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '21i <name>fs.defaultFS</name>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '22i <value>hdfs://Master:9000</value>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '23i </property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '24i <property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '25i <name>hadoop.tmp.dir</name>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '26i <value>file:/usr/local/hadoop/tmp</value>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '27i <description>Abase for other temporary directories.</description>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '28i </property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '20i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '21i <name>dfs.namenode.http-address</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '22i <value>Master:50070</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '23i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '24i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '25i <name>dfs.namenode.secondary.http-address</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '26i <value>Master:50090</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '27i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '28i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '29i <name>dfs.replication</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '30i <value>1</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '31i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '32i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '33i <name>dfs.namenode.name.dir</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '34i <value>file:/usr/local/hadoop/tmp/dfs/name</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '35i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '36i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '37i <name>dfs.datanode.data.dir</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '38i <value>file:/usr/local/hadoop/tmp/dfs/data</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '39i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            mv /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml.template /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '20i <property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '21i <name>mapreduce.framework.name</name>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '22i <value>yarn</value>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '23i </property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '24i <property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '25i <name>mapreduce.jobhistory.address</name>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '26i <value>Master:10020</value>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '27i </property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '28i <property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '29i <name>mapreduce.jobhistory.webapp.address</name>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '30i <value>Master:19888</value>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '31i </property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '18i <property> ' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '19i <name>yarn.resourcemanager.hostname</name>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '20i <value>Master</value>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '21i </property>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '22i <property>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '23i <name>yarn.nodemanager.aux-services</name>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '24i <value>mapreduce_shuffle</value>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '25i </property>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
    
            #Instantiate NameNode


            ####################################ZOO KEEPER####################################
    
            #configure /opt/zookeeper-3.4.9/conf/zoo.cfg
            mv /opt/zookeeper-3.4.9/conf/zoo_sample.cfg /opt/zookeeper-3.4.9/conf/zoo.cfg
            sed -i 's/dataDir=\/tmp\/zookeeper/dataDir=\/opt\/zookeeper-3.4.9\/data/g' /opt/zookeeper-3.4.9/conf/zoo.cfg
            echo "server.0=Master:2888:3888" >> /opt/zookeeper-3.4.9/conf/zoo.cfg
            echo "server.1=Slave1:2888:3888" >> /opt/zookeeper-3.4.9/conf/zoo.cfg
            echo "server.2=Slave2:2888:3888" >> /opt/zookeeper-3.4.9/conf/zoo.cfg
            mkdir /opt/zookeeper-3.4.9/data/
            touch /opt/zookeeper-3.4.9/data/myid
    
            echo '0'> /opt/zookeeper-3.4.9/data/myid
    
            sed -i '131i JAVA=/opt/jdk1.8.0_111/bin/java' /opt/zookeeper-3.4.9/bin/zkServer.sh
    
            ####################################HBASE#########################################
    
            #configure /opt/hbase-1.2.3/conf/hbase-env.sh
            echo "export JAVA_HOME=/opt/jdk1.8.0_111" >> /opt/hbase-1.2.3/conf/hbase-env.sh
            echo "export HBASE_MANAGES_ZK=false" >> /opt/hbase-1.2.3/conf/hbase-env.sh
    
            #configure /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '24i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '25i <name>hbase.rootdir</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '26i <value>hdfs://Master:9000/hbase</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '27i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '28i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '29i <name>hbase.cluster.distributed</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '30i <value>true</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '31i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '32i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '33i <name>hbase.zookeeper.quorum</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '34i <value>Master,Slave1,Slave2</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '35i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '36i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '37i <name>hbase.zookeeper.property.dataDir</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '38i <value>/opt/zookeeper-3.4.9/data/</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '39i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
    
            #configure /opt/hbase-1.2.3/conf/regionservers
            echo 'Master' > /opt/hbase-1.2.3/conf/regionservers
            echo 'Slave1' >> /opt/hbase-1.2.3/conf/regionservers
            echo 'Slave2' >> /opt/hbase-1.2.3/conf/regionservers
    
            echo '#!/bin/bash
    
            passwd=login_pass
    
            /usr/bin/expect << EOF
            spawn sudo ssh-keygen -P ""
            expect {
                    "Enter file in*" { 
                            send "\n";
                            exp_continue 
                    }
                    "Overwrite (y/n)*" { 
                            send "y\n" 
                    }
                    "The key*" {
                            send_user "\n"
                    }
            }
            set timeout -1
            expect eof
            EOF
    
            /usr/bin/expect << EOF
            spawn sudo ssh-copy-id root@Master
            set timeout -1
            expect {
                    "*(yes/no)?" { 
                            send "yes\n";
                            exp_continue 
                    } 
                    "*password:" { 
                    send "$passwd\n" 
                    }
                    "Now try logging into*" {
                            send_user "\n"
                    }
            }
            expect eof
            EOF
    
            /usr/bin/expect << EOF
            spawn sudo ssh-copy-id 0.0.0.0
            set timeout -1
            expect {
                    "*(yes/no)?" { 
                        send "yes\n"
                    } 
            }
            expect eof
            EOF
    
            /usr/bin/expect << EOF
            spawn sudo ssh-copy-id localhost
            expect {
                    "*(yes/no)?" { 
                        send "yes\n"
                    } 
            }
            expect eof
            EOF
    
            /usr/bin/expect << EOF
            spawn sudo ssh-copy-id root@Slave1
            set timeout -1
            expect {
                    "*(yes/no)?" { 
                        send "yes\n";
                        exp_continue 
                    } 
                    "*password:" { 
                    send "$passwd\n" 
                    }
            }
            expect eof
            EOF
    
            /usr/bin/expect << EOF
            spawn sudo ssh-copy-id root@Slave2
            set timeout -1
            expect {
                    "*(yes/no)?" { 
                        send "yes\n";
                        exp_continue 
                    } 
                    "*password:" { 
                    send "$passwd\n" 
                    }
            }
            expect eof
            EOF
    
            /usr/bin/expect << EOF
            spawn sudo /opt/hadoop-2.7.3/bin/hdfs namenode -format
            expect {
                    "Re-format*" { 
                        send "Y\n";
                    }
            }
            expect eof
            EOF
    
            /opt/hadoop-2.7.3/sbin/start-dfs.sh
    
            /opt/hadoop-2.7.3/sbin/start-yarn.sh
    
            /opt/hadoop-2.7.3/sbin/mr-jobhistory-daemon.sh start historyserver
    
            /opt/zookeeper-3.4.9/bin/zkServer.sh start
    
            ssh root@slave1 "/opt/zookeeper-3.4.9/bin/zkServer.sh start"
    
            ssh root@slave2 "/opt/zookeeper-3.4.9/bin/zkServer.sh start"
    
            /opt/hbase-1.2.3/bin/start-hbase.sh' | tee > /root/start.sh
    
            chmod 777 /root/start.sh
    
            nohup /root/start.sh > /root/start.log
    
            #echo Login password
    
            echo  "######################################################"
            echo  "######################################################"
            echo "Login as 'root' user. Default password: "login_pass
            echo  "######################################################"
            echo  "######################################################"
    
            echo "Login as 'root' user. Default password: "login_pass | tee > /root/Login_Password
    
          params:
            login_pass: { get_param: login_pass }


  server2_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: private_net }
      fixed_ips: [{ 'ip_address': 192.168.1.3}]

  server2:
    type: OS::Nova::Server
    properties:
      name: Slave1
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: server2_port }
      user_data:
        str_replace:
          template: |
            #!/bin/bash

            #change login password
    
            /usr/bin/expect << EOF
            spawn sudo passwd root
            set timeout -1
            expect {
                    "Enter new UNIX password:" { 
                            send "login_pass\n"; 
                            exp_continue 
                    }
                    "Retype new UNIX password:" { 
                            send "login_pass\n"
                    }
            }
            expect eof
            EOF
    
            #pull packages
            scp_ip=192.168.65.241
            scp_passwd=handgecloud
    
            /usr/bin/expect << EOF
            spawn scp -r root@$scp_ip:/mnt/ramdisk/opt.tgz /
            expect {
                    "*(yes/no)?" { 
                            send "yes\n"; 
                            exp_continue 
                    }
                    "*password:" { 
                            send "$scp_passwd\n"
                    }
            }
            set timeout -1
            expect eof
            EOF
    
            cd /
            pigz -d /opt.tgz
            tar -xf /opt.tar
            rm /opt.tar
    
            #vim hosts
            echo "192.168.1.2 Master master" >> /etc/hosts
            echo "192.168.1.3 Slave1 slave1" >> /etc/hosts
            echo "192.168.1.4 Slave2 slave2" >> /etc/hosts
    
            #ENV
            echo "export JAVA_HOME=/opt/jdk1.8.0_111" >> ~/.bashrc
            echo "export HADOOP_HOME=/opt/hadoop-2.7.3" >> ~/.bashrc
            echo "export HBASE_HOME=/opt/hbase-1.2.3" >> ~/.bashrc
            echo "export ZOOKEEPER_HOME=/opt/zookeeper-3.4.9" >> ~/.bashrc
            echo 'export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$JAVA_HOME/bin:$HBASE_HOME/bin:$HBASE_HOME/sbin:$ZOOKEEPER_HOME/bin:$PATH' >> ~/.bashrc
            source ~/.bashrc
            sed -i 's/JAVA_HOME=${JAVA_HOME}/JAVA_HOME=\/opt\/jdk1.8.0_111/g' /opt/hadoop-2.7.3/etc/hadoop/hadoop-env.sh
    
            ##########################HADOOP####################
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/slaves
            echo "Slave1" > /opt/hadoop-2.7.3/etc/hadoop/slaves
            echo "Slave2" >> /opt/hadoop-2.7.3/etc/hadoop/slaves
    
            touch /opt/hadoop-2.7.3/etc/hadoop/master
            echo "Master" > /opt/hadoop-2.7.3/etc/hadoop/master
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '20i <property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '21i <name>fs.defaultFS</name>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '22i <value>hdfs://Master:9000</value>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '23i </property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '24i <property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '25i <name>hadoop.tmp.dir</name>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '26i <value>file:/usr/local/hadoop/tmp</value>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '27i <description>Abase for other temporary directories.</description>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '28i </property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '20i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '21i <name>dfs.namenode.http-address</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '22i <value>Master:50070</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '23i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '24i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '25i <name>dfs.namenode.secondary.http-address</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '26i <value>Master:50090</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '27i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '28i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '29i <name>dfs.replication</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '30i <value>1</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '31i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '32i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '33i <name>dfs.namenode.name.dir</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '34i <value>file:/usr/local/hadoop/tmp/dfs/name</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '35i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '36i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '37i <name>dfs.datanode.data.dir</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '38i <value>file:/usr/local/hadoop/tmp/dfs/data</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '39i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml


            #configure /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            mv /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml.template /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '20i <property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '21i <name>mapreduce.framework.name</name>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '22i <value>yarn</value>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '23i </property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '24i <property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '25i <name>mapreduce.jobhistory.address</name>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '26i <value>Master:10020</value>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '27i </property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '28i <property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '29i <name>mapreduce.jobhistory.webapp.address</name>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '30i <value>Master:19888</value>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '31i </property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '18i <property> ' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '19i <name>yarn.resourcemanager.hostname</name>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '20i <value>Master</value>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '21i </property>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '22i <property>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '23i <name>yarn.nodemanager.aux-services</name>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '24i <value>mapreduce_shuffle</value>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '25i </property>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
    
            ####################################ZOO KEEPER####################################
    
            #configure /opt/zookeeper-3.4.9/conf/zoo.cfg
            mv /opt/zookeeper-3.4.9/conf/zoo_sample.cfg /opt/zookeeper-3.4.9/conf/zoo.cfg
            sed -i 's/dataDir=\/tmp\/zookeeper/dataDir=\/opt\/zookeeper-3.4.9\/data/g' /opt/zookeeper-3.4.9/conf/zoo.cfg
            echo "server.0=Master:2888:3888" >> /opt/zookeeper-3.4.9/conf/zoo.cfg
            echo "server.1=Slave1:2888:3888" >> /opt/zookeeper-3.4.9/conf/zoo.cfg
            echo "server.2=Slave2:2888:3888" >> /opt/zookeeper-3.4.9/conf/zoo.cfg
            mkdir /opt/zookeeper-3.4.9/data/
            touch /opt/zookeeper-3.4.9/data/myid
    
            echo '1'> /opt/zookeeper-3.4.9/data/myid
    
            sed -i '131i JAVA=/opt/jdk1.8.0_111/bin/java' /opt/zookeeper-3.4.9/bin/zkServer.sh
    
            ####################################HBASE#########################################
    
            #configure /opt/hbase-1.2.3/conf/hbase-env.sh
            echo "export JAVA_HOME=/opt/jdk1.8.0_111" >> /opt/hbase-1.2.3/conf/hbase-env.sh
            echo "export HBASE_MANAGES_ZK=false" >> /opt/hbase-1.2.3/conf/hbase-env.sh
    
            #configure /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '24i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '25i <name>hbase.rootdir</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '26i <value>hdfs://Master:9000/hbase</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '27i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '28i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '29i <name>hbase.cluster.distributed</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '30i <value>true</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '31i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '32i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '33i <name>hbase.zookeeper.quorum</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '34i <value>Master,Slave1,Slave2</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '35i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '36i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '37i <name>hbase.zookeeper.property.dataDir</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '38i <value>/opt/zookeeper-3.4.9/data/</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '39i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
    
            #configure /opt/hbase-1.2.3/conf/regionservers
            echo 'Master' > /opt/hbase-1.2.3/conf/regionservers
            echo 'Slave1' >> /opt/hbase-1.2.3/conf/regionservers
            echo 'Slave2' >> /opt/hbase-1.2.3/conf/regionservers
    
            #echo Login password
    
            echo  "######################################################"
            echo  "######################################################"
            echo "Login as 'root' user. Default password: "login_pass
            echo  "######################################################"
            echo  "######################################################"
    
            echo "Login as 'root' user. Default password: "login_pass | tee > /root/Login_Password
    
          params:
            login_pass: { get_param: login_pass }

  server3_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: private_net }
      fixed_ips: [{ 'ip_address': 192.168.1.4}]

  server3:
    type: OS::Nova::Server
    properties:
      name: Slave2
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: server3_port }
      user_data:
        str_replace:
          template: |
            #!/bin/bash

            #change login password
    
            /usr/bin/expect << EOF
            spawn sudo passwd root
            set timeout -1
            expect {
                    "Enter new UNIX password:" { 
                            send "login_pass\n"; 
                            exp_continue 
                    }
                    "Retype new UNIX password:" { 
                            send "login_pass\n"
                    }
            }
            expect eof
            EOF
    
            #pull packages
            scp_ip=192.168.65.241
            scp_passwd=handgecloud
    
            /usr/bin/expect << EOF
            spawn scp -r root@$scp_ip:/mnt/ramdisk/opt.tgz /
            expect {
                    "*(yes/no)?" { 
                            send "yes\n"; 
                            exp_continue 
                    }
                    "*password:" { 
                            send "$scp_passwd\n"
                    }
            }
            set timeout -1
            expect eof
            EOF
    
            cd /
            pigz -d /opt.tgz
            tar -xf /opt.tar
            rm /opt.tar
    
            #vim hosts
            echo "192.168.1.2 Master master" >> /etc/hosts
            echo "192.168.1.3 Slave1 slave1" >> /etc/hosts
            echo "192.168.1.4 Slave2 slave2" >> /etc/hosts
    
            #ENV
            echo "export JAVA_HOME=/opt/jdk1.8.0_111" >> ~/.bashrc
            echo "export HADOOP_HOME=/opt/hadoop-2.7.3" >> ~/.bashrc
            echo "export HBASE_HOME=/opt/hbase-1.2.3" >> ~/.bashrc
            echo "export ZOOKEEPER_HOME=/opt/zookeeper-3.4.9" >> ~/.bashrc
            echo 'export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$JAVA_HOME/bin:$HBASE_HOME/bin:$HBASE_HOME/sbin:$ZOOKEEPER_HOME/bin:$PATH' >> ~/.bashrc
            source ~/.bashrc
            sed -i 's/JAVA_HOME=${JAVA_HOME}/JAVA_HOME=\/opt\/jdk1.8.0_111/g' /opt/hadoop-2.7.3/etc/hadoop/hadoop-env.sh
    
            ##########################HADOOP####################
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/slaves
            echo "Slave1" > /opt/hadoop-2.7.3/etc/hadoop/slaves
            echo "Slave2" >> /opt/hadoop-2.7.3/etc/hadoop/slaves
    
            touch /opt/hadoop-2.7.3/etc/hadoop/master
            echo "Master" > /opt/hadoop-2.7.3/etc/hadoop/master
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '20i <property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '21i <name>fs.defaultFS</name>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '22i <value>hdfs://Master:9000</value>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '23i </property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '24i <property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '25i <name>hadoop.tmp.dir</name>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '26i <value>file:/usr/local/hadoop/tmp</value>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '27i <description>Abase for other temporary directories.</description>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
            sed -i '28i </property>' /opt/hadoop-2.7.3/etc/hadoop/core-site.xml
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '20i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '21i <name>dfs.namenode.http-address</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '22i <value>Master:50070</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '23i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '24i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '25i <name>dfs.namenode.secondary.http-address</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '26i <value>Master:50090</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '27i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '28i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '29i <name>dfs.replication</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '30i <value>1</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '31i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '32i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '33i <name>dfs.namenode.name.dir</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '34i <value>file:/usr/local/hadoop/tmp/dfs/name</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '35i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '36i <property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '37i <name>dfs.datanode.data.dir</name>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '38i <value>file:/usr/local/hadoop/tmp/dfs/data</value>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml
            sed -i '39i </property>' /opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml


            #configure /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            mv /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml.template /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '20i <property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '21i <name>mapreduce.framework.name</name>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '22i <value>yarn</value>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '23i </property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '24i <property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '25i <name>mapreduce.jobhistory.address</name>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '26i <value>Master:10020</value>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '27i </property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '28i <property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '29i <name>mapreduce.jobhistory.webapp.address</name>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '30i <value>Master:19888</value>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
            sed -i '31i </property>' /opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml
    
            #configure /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '18i <property> ' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '19i <name>yarn.resourcemanager.hostname</name>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '20i <value>Master</value>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '21i </property>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '22i <property>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '23i <name>yarn.nodemanager.aux-services</name>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '24i <value>mapreduce_shuffle</value>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
            sed -i '25i </property>' /opt/hadoop-2.7.3/etc/hadoop/yarn-site.xml
    
            ####################################ZOO KEEPER####################################
    
            #configure /opt/zookeeper-3.4.9/conf/zoo.cfg
            mv /opt/zookeeper-3.4.9/conf/zoo_sample.cfg /opt/zookeeper-3.4.9/conf/zoo.cfg
            sed -i 's/dataDir=\/tmp\/zookeeper/dataDir=\/opt\/zookeeper-3.4.9\/data/g' /opt/zookeeper-3.4.9/conf/zoo.cfg
            echo "server.0=Master:2888:3888" >> /opt/zookeeper-3.4.9/conf/zoo.cfg
            echo "server.1=Slave1:2888:3888" >> /opt/zookeeper-3.4.9/conf/zoo.cfg
            echo "server.2=Slave2:2888:3888" >> /opt/zookeeper-3.4.9/conf/zoo.cfg
            mkdir /opt/zookeeper-3.4.9/data/
            touch /opt/zookeeper-3.4.9/data/myid
    
            echo '2'> /opt/zookeeper-3.4.9/data/myid
    
            sed -i '131i JAVA=/opt/jdk1.8.0_111/bin/java' /opt/zookeeper-3.4.9/bin/zkServer.sh
    
            ####################################HBASE#########################################
    
            #configure /opt/hbase-1.2.3/conf/hbase-env.sh
            echo "export JAVA_HOME=/opt/jdk1.8.0_111" >> /opt/hbase-1.2.3/conf/hbase-env.sh
            echo "export HBASE_MANAGES_ZK=false" >> /opt/hbase-1.2.3/conf/hbase-env.sh
    
            #configure /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '24i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '25i <name>hbase.rootdir</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '26i <value>hdfs://Master:9000/hbase</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '27i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '28i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '29i <name>hbase.cluster.distributed</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '30i <value>true</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '31i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '32i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '33i <name>hbase.zookeeper.quorum</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '34i <value>Master,Slave1,Slave2</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '35i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '36i <property>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '37i <name>hbase.zookeeper.property.dataDir</name>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '38i <value>/opt/zookeeper-3.4.9/data/</value>' /opt/hbase-1.2.3/conf/hbase-site.xml
            sed -i '39i </property>' /opt/hbase-1.2.3/conf/hbase-site.xml
    
            #configure /opt/hbase-1.2.3/conf/regionservers
            echo 'Master' > /opt/hbase-1.2.3/conf/regionservers
            echo 'Slave1' >> /opt/hbase-1.2.3/conf/regionservers
            echo 'Slave2' >> /opt/hbase-1.2.3/conf/regionservers
    
            #echo Login password
    
            echo  "######################################################"
            echo  "######################################################"
            echo "Login as 'root' user. Default password: "login_pass
            echo  "######################################################"
            echo  "######################################################"
    
            echo "Login as 'root' user. Default password: "login_pass | tee > /root/Login_Password
    
          params:
            login_pass: { get_param: login_pass }

  server4_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: private_net }
      fixed_ips: [{ 'ip_address': 192.168.1.5}]

  server4_floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: { get_param: public_net }
      port_id: { get_resource: server4_port }

  server4:
    type: OS::Nova::Server
    properties:
      name: dev1
      image: { get_param: image_desktop }
      flavor: { get_param: flavor_desktop }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: server4_port }
      user_data:
        str_replace:
          template: |
            #!/bin/bash

            /usr/bin/expect << EOF
            spawn sudo passwd root
            set timeout -1
            expect {
                    "Enter new UNIX password:" { 
                            send "login_pass\n"; 
                            exp_continue 
                    }
                    "Retype new UNIX password:" { 
                            send "login_pass\n"
                    }
            }
            expect eof
            EOF
    
            sed -i "s/localhost/localhost dev1/g" /etc/hosts
    
            echo "Login as 'root' user. Default password: "login_pass | tee > /root/Login_Password
    
          params:
            login_pass: { get_param: login_pass }

  server5_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: private_net }
      fixed_ips: [{ 'ip_address': 192.168.1.6}]

  server5_floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: { get_param: public_net }
      port_id: { get_resource: server5_port }

  server5:
    type: OS::Nova::Server
    properties:
      name: dev2
      image: { get_param: image_desktop }
      flavor: { get_param: flavor_desktop }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: server5_port }
      user_data:
        str_replace:
          template: |
            #!/bin/bash

            /usr/bin/expect << EOF
            spawn sudo passwd root
            set timeout -1
            expect {
                    "Enter new UNIX password:" { 
                            send "login_pass\n"; 
                            exp_continue 
                    }
                    "Retype new UNIX password:" { 
                            send "login_pass\n"
                    }
            }
            expect eof
            EOF
    
            sed -i "s/localhost/localhost dev2/g" /etc/hosts
    
            echo "Login as 'root' user. Default password: "login_pass | tee > /root/Login_Password
    
          params:
            login_pass: { get_param: login_pass }

  server6_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_resource: private_net }
      fixed_ips: [{ 'ip_address': 192.168.1.7}]

  server6_floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: { get_param: public_net }
      port_id: { get_resource: server6_port }

  server6:
    type: OS::Nova::Server
    properties:
      name: dev3
      image: { get_param: image_desktop }
      flavor: { get_param: flavor_desktop }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: server6_port }
      user_data:
        str_replace:
          template: |
            #!/bin/bash

            /usr/bin/expect << EOF
            spawn sudo passwd root
            set timeout -1
            expect {
                    "Enter new UNIX password:" { 
                            send "login_pass\n"; 
                            exp_continue 
                    }
                    "Retype new UNIX password:" { 
                            send "login_pass\n"
                    }
            }
            expect eof
            EOF
    
            sed -i "s/localhost/localhost dev3/g" /etc/hosts
    
            echo "Login as 'root' user. Default password: "login_pass | tee > /root/Login_Password
    
          params:
            login_pass: { get_param: login_pass }

outputs:
  server1:
    description: Login password of Master for user 'root'
    value: {get_param: login_pass}
  server1_private_ip:
    description: IP address of Master in private network
    value: { get_attr: [ server1, first_address ] }
  server1_public_ip:
    description: Floating IP address of Master in public network
    value: { get_attr: [ server1_floating_ip, floating_ip_address ] }

  server2:
    description: Login password of Slave1 for user 'root'
    value: {get_param: login_pass}
  server2_private_ip:
    description: IP address of Slave1 in private network
    value: { get_attr: [ server2, first_address ] }

  server3:
    description: Login password of Slave2 for user 'root'
    value: {get_param: login_pass}
  server3_private_ip:
    description: IP address of Slave2 in private network
    value: { get_attr: [ server3, first_address ] }

  server4:
    description: Login password of dev1 for user 'ubuntu'
    value: {get_param: login_pass}
  server4_private_ip:
    description: IP address of dev1 in private network
    value: { get_attr: [ server4, first_address ] }
  server4_public_ip:
    description: Floating IP address of ubuntu in public network
    value: { get_attr: [ server4_floating_ip, floating_ip_address ] }

  server5:
    description: Login password of dev2 for user 'ubuntu'
    value: {get_param: login_pass}
  server5_private_ip:
    description: IP address of dev2 in private network
    value: { get_attr: [ server5, first_address ] }
  server5_public_ip:
    description: Floating IP address of ubuntu in public network
    value: { get_attr: [ server5_floating_ip, floating_ip_address ] }

  server6:
    description: Login password of dev3 for user 'ubuntu'
    value: {get_param: login_pass}
  server6_private_ip:
    description: IP address of dev3 in private network
    value: { get_attr: [ server6, first_address ] }
  server6_public_ip:
    description: Floating IP address of ubuntu in public network
    value: { get_attr: [ server6_floating_ip, floating_ip_address ] }
```