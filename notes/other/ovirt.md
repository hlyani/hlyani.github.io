# Ovirt 相关

## 一、介绍

> oVirt是一个开源服务，是部署在操作系统（CentOS、Red Hat Enterprise Linux）上的桌面虚拟化平台。

##### 1、oVirt-engine

> 虚拟机允许你配置你的网络，存储，节点还有镜像，虚拟机也提供了命令行工具ovirt-engine-cli和很实用的API（ovirt-engine-api），包含了python包装器，这个wrapper可以允许开发者整合功能到第三方的shell脚本中管理。

##### 2、VDSM

> 这个虚拟桌面和服务管理守护进程运行在oVirt的管理节点上，允许oVirt远程的部署，开始，停止monitor端的机器。

##### 3、oVirt-node

> 虚拟节点仅是一个运行在虚拟机上的操作系统，他也可以把一个标准发行版的linux转换成一个节点，这个节点可以通过 ovirt-engine管理，通过VDSM和其他依赖安装。

##### 4、dwh and reports

> 对于ovirt-engine，报告和数据仓库的这个组件是可选择的，并且是分别分装和开发的。

##### 5、基本概念

* 数据中心：数据中心是所有物理资源和逻辑资源的最大容器单位，它是所有虚拟机，存储，和网络的集合。

* 集群： 一个集群指的是物理主机的集合，在这里，主机可以看做是虚拟机的资源池。在同一个集群中的主机共享相同的网络和存储架构，而且集群中的虚拟机可以在属于该集群的不同的主机之间迁移。

* 逻辑网络：逻辑网络是物理网络在逻辑上的表示，它把网络负载根据管理流量，主机流量，存储流量和虚拟机的流量分组，从而更好的实现网络的管理和流量的分离。

* 主机：主机就是物理上的服务器，虚拟机在主机上运行。一个主机可以运行一个或者多个虚拟机，正如上面提到的，是主机组成了集群。在一个集群中的虚拟机可以在不同的主机间迁移。

* 存储池：存储池是一个逻辑上的概念，包含同一种存储类型的仓库，比如 iSCSI，Fibre Channel，NFS，或者 POSIX。每一种存储池都可以包含几个同类型的存储域，用来存储虚拟机镜像，ISO 文件，或者是用来导入/导出存储域。

* 虚拟机：虚拟机就像实际的机器一样，有自己的硬件(CPU，内存等)，包含操作系统和一系列应用软件。 EayunOS 系统中的虚拟机有两种: 虚拟桌面和虚拟服务器。多个同样的虚拟机能同时快速的在一个池里面创建。注意，虚拟机的创建，管理，删除等操作只能被超级用户和授予相关权限的用户执行。

* 模板：模板是一种虚拟机模型，这种模型预先定义了虚拟机的很多内容，比如操作系统等。通过模板，可以在简单的一个操作中以最快的方式创建大量的虚拟机。

* 虚拟机池：虚拟机池是指一组相同的虚拟机的集合。对一些特定的目的，虚拟机池很有用。比如不同部门虚拟机使用的划分，一个池给市场部门用，另一个池给研发部门用，等等。

* 快照：快照是指某一个时间点虚拟机操作系统的所有内容的一个状态。快照有很多用途，比如在升级虚拟机或者修改虚拟机内容的时候，可以建立一个快照，当升级完系统出问题的时候，可以用快照恢复到之前的状态。

* 用户类型：系统支持不同级别的管理员和用户权限。系统管理员管理物力资源，比如数据中心，主机，存储资源等。是系统管理员建立的虚拟机池和虚拟机，用户具有访问这些虚拟机的能力。

* 事件和监控：报警，警告和其它关于系统的通知可以帮助系统管理员更好的了解整个系统的性能和资源的状态。

* 报表：从基于 JasperReports 的报表模块，或是数据库产生需要的报表。用户也可以使用任何支持 SQL 语句的查询工具查询包含主机，虚拟机和存储等的监控数据。

## 二、系统要求

| 资源              | 最小要求       |
| ----------------- | -------------- |
| CPU               | 2核            |
| Memory            | >4GB           |
| Hard Disk         | >25G           |
| Network Interface | >1张 ， >1Gbps |

## 三、安装

[install ovirt](https://www.ovirt.org/documentation/install-guide/chap-Installing_oVirt.html)

##### 1、安装ovirt-engine相关软件

```
yum -y install http://resources.ovirt.org/pub/yum-repo/ovirt-release43.rpm
yum -y update
yum -y install nfs-utils ovirt-engine
```

##### 2、配置ovirt-engine

```
engine-setup
```

##### 3、连接到管理员界面

> 在浏览器输入地址，https://your-manager-fqdn/ovirt-engine，第一次登陆用户名admin。

```
# vim /etc/ovirt-engine/engine.conf.d/99-custom-sso-setup.conf
SSO_ALTERNATE_ENGINE_FQDNS="alias1.example.com alias2.example.com"

admin@internal
http://myovirt:80/ovirt-engine
https://myovirt:443/ovirt-engine
```

##### 4、添加存储

```
systemctl start rpcbind
systemctl enable rpcbind

mkdir /iso /data /import_export
chown -R vdsm:kvm /iso 
chown -R vdsm:kvm /data
chown -R vdsm:kvm /import_export

在exports中添加相关内容
vim /etc/exports
vim /etc/exports.d/ovirt-engine-iso-domain.exports

/iso            *(rw,sync,no_subtree_check,all_squash,anonuid=36,anongid=36)
/data           *(rw,sync,no_subtree_check,all_squash,anonuid=36,anongid=36)
/import_export  *(rw,sync,no_subtree_check,all_squash,anonuid=36,anongid=36)

systemctl restart rpcbind
systemctl restart nfs

exportfs -r

service ovirt-engine restart
```

##### 5、上传iso镜像

```
engine-iso-uploader -i iso upload /root/cirros-0.3.4-x86_64-disk.iso
```

##### 6、查看ovirt-engine日志

```
tailf /var/log/ovirt-engine/engine.log
```

##### 7、ovirt node

> 安装完oVirt Engine服务后，就可以安装节点主机来运行虚拟机了。在oVirt架构中，你可以使用 oVirt Node, Fedora 或者 CentOS 做为节点主机的操作系统。

> centos7 node 修改hosts文件engine加进去

##### 8、使用virsh list

```
virsh list

vdsm@ovirt
shibboleth

hosted-engine --help
```

## 四、ovirt编译

#### （一）、使用rpm方式编译

```
wget http://resources.ovirt.org/pub/ovirt-4.1/rpm/el7Workstation/SRPMS/ovirt-engine-4.1.4.2-1.el7.centos.src.rpm
 
//axel -n 4 http://resources.ovirt.org/pub/ovirt-4.1/rpm/el7Workstation/SRPMS/ovirt-engine-4.1.4.2-1.el7.centos.src.rpm
 
rpm -i ovirt-engine-4.1.4.2-1.el7.centos.src.rpm
 
vim ~/rpmbuild/SPECS/ovirt-engine.spec
 
rpm -ql ovirt-engine |grep jar
 
yum -y install rpm-build
 
rpmbuild -ba ~/rpmbuild/SPECS/ovirt-engine.spec //根据 spec 文件同时构建二进制 RPM 和源代码 RPM 
 
//rpm -U
```

> Building locales requires more than 10240 available file descriptors, currently 1024

```
ulimit -n 10240
```

> [ERROR] OutOfMemoryError: Increase heap size or lower gwt.jjs.maxThreads

```
cd /root/rpmbuild/SOURCES

tar -zxvf ovirt-engine-4.1.4.2.tar.gz

vim frontend/webadmin/modules/pom.xml

<gwt.jvmArgs>-Xms8g -Xmx8g -XX:MaxDirectMemorySize=4096m</gwt.jvmArgs>  //Two JVM options are often used to tune JVM heap size: -Xmx for maximum heap size, and -Xms for initial heap size.

<gwt-plugin.extraJvmArgs>
  -Dgwt.jjs.permutationWorkerFactory=com.google.gwt.dev.ThreadedPermutationWorkerFactory \
  -Dgwt.jjs.maxThreads=1 \
  -Djava.io.tmpdir="${project.build.directory}/tmp" \
  -Djava.util.prefs.systemRoot="${project.build.directory}/tmp" \
  -Djava.util.prefs.userRoot="${project.build.directory}/tmp" \
  -Djava.util.logging.config.class=org.ovirt.engine.ui.gwtextension.JavaLoggingConfig \
  ${gwt.jvmArgs}
</gwt-plugin.extraJvmArgs>

rm -rf ovirt-engine-4.1.4.2.tar.gz

tar -zcvf ovirt-engine-4.1.4.2.tar.gz *

rpmbuild -ba ~/rpmbuild/SPECS/ovirt-engine.spec //重新编译 
```

#### （二）、搭建环境并使用build方式编译

#####  一、安装构建环境

###### 1、安装snapshot库

```
yum -y install http://resources.ovirt.org/pub/yum-repo/ovirt-release42.rpm
```

###### 2、安装编译依赖包

```
yum -y install mailcap openssl m2crypto python-psycopg2 python-cheetah python-daemon libxml2-python unzip ovirt-host-deploy ovirt-setup-lib git maven postgresql-server java-1.8.0-openjdk-devel wget python-pip
```

###### 3、检查 JDK

使用“alternatives”命令验证javac是否已经指向已安装的Jdk1.8路径。

```
alternatives --display javac
```

如果javac没有指向正确的目录，那么可以使用以下命令进行设置。

```
alternatives --set javac /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/bin/javac
```

###### 4.检查 maven	

验证Maven的环境变量已经设置成功

```
mvn -version
```

Maven 设置

设置 ~/.m2/ directory 库。

将下列内容拷贝到配置settings文件中。

```
cat > ~/.m2/settings.xml <<"EOT"

<settings xmlns="http://maven.apache.org/POM/4.0.0"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                         http://maven.apache.org/xsd/settings-1.0.0.xsd">
     <activeProfiles>
          <activeProfile>oVirtEnvSettings</activeProfile>
     </activeProfiles>
     <profiles>
          <profile>
               <id>oVirtEnvSettings</id>
               <properties>                               
                    <jbossHome>${env.JBOSS_HOME}</jbossHome>
                    <JAVA_HOME>${env.JAVA_HOME}</JAVA_HOME>
               </properties>
          </profile>
      </profiles>
</settings>

EOT
```

备注：

一定要配置以上maven配置文件，否则在后续编译中会出错。请确保jdk的环境变量设置正确。

在环境变量中设置以上需要的环境变量方法为:在用户的家目录中将以下环境变量添加到/etc/profile文件中。

```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.el7_4.x86_64/
export JBOSS_HOME=/usr/share/jboss-as
```

##### 二、安装JBoss AS

```
cd /usr/share
wget http://download.jboss.org/jbossas/7.1/jboss-as-7.1.1.Final/jboss-as-7.1.1.Final.tar.gz
tar -zxvf jboss-as-7.1.1.Final.tar.gz --no-same-owner
ln -s /usr/share/jboss-as-7.1.1.Final /usr/share/jboss-as
su - -c 'chmod -R 777 /usr/share/jboss-as'
```

校验是否安装成功

```
/usr/share/jboss-as/bin/standalone.sh
```

请确保对于$JBOSS_HOME/standalone/deployments文件夹有写权限，该目录是ovirt-engine的部署目录。

TroubleShooting

###### 1.下面是一些有用的JAVA_OPTS的设置，可以添加到standalone.conf脚本中：

```
· -Xmx512m - maximum Java heap size of 512m

· -Xdebug - include debugging
```

###### 2.在运行时添加-b 0.0.0.0用于绑定所有的IP

###### 3.确定8080或者8009端口未被占用。其他可能需要的端口号有:8443/8083/1090/4457

###### 4.JBoss会绑定主机名称，确认该主机名称已经被添加到/etc/hosts目录下。

###### 5.如果部署后的文件并不是最新的代码，可以使用下面的语句移除已经先部署的代码。

```
# > cd $JBOSS_HOME/standalone

# > rm -rf deployments/engine.ear 

# > rm -rf deployments/engine.ear.deployed 

# > rm -rf tmp

# > rm -rf data (should be done only in development environment)    

# > # Change the JBOSS_HOME environment variable to the new location

# > su - -c 'chmod -R 777 /usr/share/jboss-as'

# > # Change the Jboss home in ~/.m2/settings.xml file to point to the new location
```

##### 三、安装PostgreSQL

初始化数据库

```
service postgresql initdb
chkconfig postgresql on
service postgresql restart
```


修改数据库连接

```
vim /var/lib/pgsql/data/pg_hba.conf
```

```
默认：
# TYPE  DATABASE    USER        CIDR-ADDRESS          METHOD
# "local" is for Unix domain socket connections only
local   all         all                               peer
# IPv4 local connections:
host    all         all         127.0.0.1/32          ident
# IPv6 local connections:
host    all         all         ::1/128               ident

做如下修改：【ps:如果后期有问题，可将ident,password,password都修改为trust】
# TYPE  DATABASE    USER        CIDR-ADDRESS          METHOD
# "local" is for Unix domain socket connections only
local   all         all                               ident
# IPv4 local connections:
host    all         all         127.0.0.1/32          password
# IPv6 local connections:
host    all         all         ::1/128               password
```

重启：

```
service postgresql restart
```

创建数据库

```
su - postgres -c "psql -d template1 -c \"create user engine password 'engine';\""
```

```
# CREATE ROLE

su - postgres -c "psql -d template1 -c \"create database engine owner engine template template0 encoding 'UTF8' lc_collate 'en_US.UTF-8' lc_ctype 'en_US.UTF-8';\""

# CREATE DATABASE
```

##### 四、编译oVirt-engine源码（切换至普通用户模式）

1. ###### 获取源码

```
git clone git://gerrit.ovirt.org/ovirt-engine
```
编译（必须在ovirt-engine目录下）

```
make install-dev PREFIX="$HOME/ovirt-engine"
```

##### 五、Setup安装（切换至普通用户模式

```
HOME/ovirt-engine/bin/engine-setup
```

##### 六、启动服务（切换至普通用户模式）

```
HOME/ovirt-engine/share/ovirt-engine/services/ovirt-engine/ovirt-engine.py start
```

##### 七、修改代码后

重新安装： 

```
make install-dev PREFIX="$HOME/ovirt-engine" 
```

重新编译： 

```
make clean install-dev PREFIX="$HOME/ovirt-engine" 
```

重新编译指定模块： 

```
make clean install-dev PREFIX=$HOME/ovirt-engine \ 
EXTRA_BUILD_FLAGS="-pl org.ovirt.engine.core:webadmin" 
```


GWT调试模式：

```
make install-dev PREFIX="$HOME/ovirt-engine" 
make gwt-debug DEBUG_MODULE=<module> 
While <module> is webadmin or userportal-gwtp. 
```

##### (三)、编译开发

#### 添加ovirt42源

```
yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release42.rpm
touch /etc/yum.repos.d/ovirt-snapshots.repo

[ovirt-snapshots]
name=local
baseurl=http://resources.ovirt.org/pub/ovirt-master-snapshot/rpm/el$releasever
enabled=1
gpgcheck=0
priority=10

[ovirt-snapshots-static]
name=local
baseurl=http://resources.ovirt.org/pub/ovirt-master-snapshot-static/rpm/el$releasever
enabled=1
gpgcheck=0
priority=10
```

#### 安装第三方包

```
yum -y install git java-devel maven openssl postgresql-server \
     m2crypto python-psycopg2 python-cheetah python-daemon libxml2-python \
     unzip pyflakes python-pep8 python-docker-py mailcap python-jinja2 \
     python-dateutil bind-utils
```

#### WildFly 8.2 for oVirt 3.6+ development

```
yum -y install ovirt-engine-wildfly ovirt-engine-wildfly-overlay
```

#### JBoss 7.1.1 for backporting changes to oVirt 3.5

```
yum -y install ovirt-engine-jboss-as
```

#### 安装ovirt包

```
yum install ovirt-host-deploy ovirt-setup-lib ovirt-js-dependencies
```

#### Make sure openjdk is the java preferred:

```
alternatives --config java
alternatives --config javac
```

#### 数据库配置

```
#postgresql-setup -D /data/psql initdb
postgresql-setup initdb

vim /var/lib/pgsql/data/pg_hba.conf
...
host    all             all             192.168.21.0/24            trust
...

systemctl restart postgresql.service
systemctl enable postgresql.service

su - postgres -c "psql -d template1 -c \"create user engine password 'engine';\""
su - postgres -c "psql -d template1 -c \"create database engine owner engine template template0 encoding 'UTF8' lc_collate 'en_US.UTF-8' lc_ctype 'en_US.UTF-8';\""
```

#### 编译

```
mkdir -p "$HOME"
cd "$HOME"
git clone git://gerrit.ovirt.org/ovirt-engine

cd ovirt-engine
make install-dev PREFIX="$HOME/ovirt-engine"

If WildFly 8.2 should be used, then it's required to manually setup ovirt-engine-wildfly-overlay using following command:

echo "ENGINE_JAVA_MODULEPATH="/usr/share/ovirt-engine-wildfly-overlay/modules:${ENGINE_JAVA_MODULEPATH}"" \
  > $PREFIX/etc/ovirt-engine/engine.conf.d/20-setup-jboss-overlay.conf
```

#### 运行

```
$HOME/ovirt-engine/bin/engine-setup
```

#### ovirt运行起来过后，开启ovirt-engine服务

```
$HOME/ovirt-engine/share/ovirt-engine/services/ovirt-engine/ovirt-engine.py start
```

## 五、定制化图标

##### 1、ovirt_top_logo.png

```
/usr/share/ovirt-engine/branding/ovirt.brand/images/ovirt_top_logo.png
```

##### 2、/usr/share/ovirt-engine/branding/ovirt.brand/welcome_page.template

```
<script>
    //document.location = "webadmin/?locale={userLocale}";
    document.location = "webadmin/?locale=zh_CN";
</script>
```

##### 3、vim backend/manager/modules/welcome/src/main/webapp/WEB-INF/ovirt-engine.jsp

```
<html class="obrand_background" style="display:none">

//document.write('<span class="version-text"><fmt:message key="obrand.welcome.version"><fmt:param value="${requestScope[\'version\']}" /> </fmt:message></span>')
```

##### 4、common.css

```
.obrand_loginPageLogoImage {
    /background-image: url(images/ovirt_top_right_logo.png);/
```

.obrand_loginPageLogoImage {
    /*background-image: url(images/ovirt_top_right_logo.png);*/

##### 5、branding/ovirt.brand/external_resources.properties

```
//注释所有
```

##### 6、vim messages_zh_CN.properties

```
oVirt -> tmpCloud
```

## 六、add license

>  本地开发环境打包，打包结果如enginesso.war文件夹，将目录下所有文件拷贝至服务器engine部署目录，例如/usr/share/ovirt-engine/engine.ear/enginesso.war/，然后重启服务即可 

```
org.ovirt.engine.core.sso.servlets.InteractiveAuthServlet

/**
             * 注册码验证开始
             */
            boolean checkRes = true;
            try {
                String filepath = "/etc/ovirt-engine/license";
                File file = new File(filepath);
                if(!file.exists()){
                    response.sendRedirect(request.getContextPath() + "/fileNotException.html");
                    return;
                }
                BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream(filepath)));
                String firstLine = reader.readLine();

                String reg = firstLine.substring(0,36);
                String mac2 = firstLine.substring(36);

                String mac = getStr1(reg) + mac2;
                String regCode = getStr2(reg);

                if(!(regCode.contains("base") | regCode.contains("super"))){
                    log.error("error user");
                    response.sendRedirect(request.getContextPath() + "/roleException.html");
                    return;
                }else {
                    Long timeString = Long.valueOf(regCode.substring(0,10));
                    Long nowtime = System.currentTimeMillis()/1000;
                    if(nowtime>timeString){
                        log.error("error time");
                        response.sendRedirect(request.getContextPath() + "/timeException.html");
                        return;
                    }
                }
                if(regCode.contains("base")){
                    String machineCodePath = "/etc/ovirt-engine/product_uuid";
                    reader = new BufferedReader(new InputStreamReader(new FileInputStream(machineCodePath)));
                    String machineCode = reader.readLine().replace("/","");

                    String publicKey = RSAUtils.loadKeyByFile("/etc/ovirt-engine/license_public_key.pem");

                    checkRes = RSAUtils.verify(BaseEncoding.base64().decode(publicKey), machineCode.getBytes(), BaseEncoding.base64().decode(mac));
                    if (checkRes == false){
                        response.sendRedirect(request.getContextPath() + "/systemStand.html");
                        return;
                    }
                }
                reader.close();
            } catch (Exception e) {
                log.error(e.getMessage());
                log.error("-------------解析license信息发生异常--------------");
                response.sendRedirect(request.getContextPath() + "/fileException.html");
                return;
            }
```

## 七、hosted-engine

<https://www.ovirt.org/develop/release-management/features/integration/heapplianceflow/>

<https://www.ovirt.org/blog/2016/07/Manage-Your-Hosted-Engine-Hosts-Deployment/>

##### 1、安装部署hosted-engine

```
yum -y install http://resources.ovirt.org/pub/yum-repo/ovirt-release41.rpm

==== nfs搭建 ====

systemctl restart nfs

mkdir /data
chown -R vdsm:kvm /data

echo "/data *(rw,sync,no_subtree_check,all_squash,anonuid=36,anongid=36)"|tee > /etc/exports

exportfs -r 

showmount -e

yum install ovirt-hosted-engine-setup

hosted-engine --deploy
```

##### 2、在页面添加额外的host（主机）

```
先添加一个存储域

主机 → 新建 → 承载的引擎 → 选择承载引擎部署操作 → 部署
```

## 八、用户管理

> ovirt存在两种用户域：本地域和外部域。平台安装过程中会创建一个“internal domain”默认本地域和默认用户admin。在本地域创建的用户为本地用户，在外部域创建的为目录用户
> 可以通过ovirt-aaa-jdbc-tool来创建和管理本地用户（aaa是Authentication, Authorization and Accounting的缩写）

##### 1、外部域

1、搭建Active Directory和DNS 

2、Active Directory 用户和计算机 → ad.com → Users → 右键（新建） → 用户 

> 注意： 1、非管理账户 2、时钟同步

ovirt可以连接外部目录服务器添加外部域， 支持的目录服务器包括：

- 389ds
- 389ds RFC-2307 Schema
- Active Directory
- FreeIPA
- Red Hat Identity Management (IdM)
- Novell eDirectory RFC-2307 Schema
- OpenLDAP RFC-2307 Schema
- OpenLDAP Standard Schema
- Oracle Unified Directory RFC-2307 Schema
- RFC-2307 Schema (Generic)
- Red Hat Directory Server (RHDS)
- Red Hat Directory Server (RHDS) RFC-2307 Schema
- iPlanet

>  ovirt使用插件ovirt-engine-extension-aaa-ldap来配置外部域。
> 参考<https://superlc320.gitbooks.io/samba-ldap-centos7/ldap_+_centos_7.html> 配置OpenLDAP服务。
> 运行

```
ovirt-engine-extension-aaa-ldap-setup
```

开启配置向导

> 参考 <https://gi thub.com/oVirt/ovirt-engine-extension-aaa-ldap> 
> 将/ad/. (Active Directory) 或者 examples/simple/. 下的文件复制到 /etc/ovirt-engine 并更改文件名和配置中的vars.* (用户名及密码等)
> 登录测试

```
# ovirt-engine-extensions-tool aaa login-user \
        --profile=@PROFILE@ --user-name=@USER@

   Replace:
    - @PROFILE@ with authn ovirt.engine.aaa.authn.profile.name.
    - @USER@ with user you want to test.
```

##### 2、本地域

> ovirt使用插件ovirt-engine-extension-aaa-jdbc来添加和配置本地域。当自己添加aaa-jdbc文件时需要配置postgresql作为aaa-jdbc插件的数据库。
> 参考 <https://github.com/oVirt/ovirt-engine-extension-aaa-jdbc> 

```
1.    su - postgres -c "psql -d template1 << __EOF__
    create user DB_USER password 'DB_PASSWORD';
    create database DB_NAME owner DB_USER template template0
        encoding 'UTF8' lc_collate 'en_US.UTF-8' lc_ctype 'en_US.UTF-8';
   __EOF__
   "
2.   vi /var/lib/pgsql/data/pg_hba.conf (please
   replace DB_NAME and DB_USER with real values):添加

     host    DB_NAME    DB_USER    0.0.0.0/0       md5
     host    DB_NAME    DB_USER    ::0/0           md5

   These line must be located prior to following lines:

     host    all        all        127.0.0.1/32    ident
     host    all        all        ::1/128         ident
重启postgresql

3. PGPASSWORD="DB_PASSWORD" \
       /usr/share/ovirt-engine-extension-aaa-jdbc/dbscripts/schema.sh \
       -s DB_HOST \
       -p DB_PORT \
       -d DB_NAME \
       -u DB_USER \
       -c apply

4. Setup AAA profile
   cp /usr/share/ovirt-engine-extension-aaa-jdbc/examples/extension.d/authn.properties        /etc/ovirt-engine/extensions.d/PROFILE-authn.properties
  cp /usr/share/ovirt-engine-extension-aaa-jdbc/examples/extension.d/authz.properties        /etc/ovirt-engine/extensions.d/PROFILE-authz.properties
  cp /usr/share/ovirt-engine-extension-aaa-jdbc/examples/aaa/profile.properties        /etc/ovirt-engine/aaa/PROFILE.properties
并编辑这三个配置文件中的变量
重启ovirt-engine

5. 添加和管理用户
ovirt-aaa-jdbc-tool --db-config=/etc/ovirt-engine/aaa/PROFILE.properties
```

ovirt 通过ovirt-aaa-jdbc-tool创建和管理本地用户/组

<https://access.redhat.com/documentation/zh-cn/red_hat_virtualization/4.1/html/administration_guide/sect-administering_user_tasks_from_the_commandline>

```
创建一个新用户账户 ovirt-aaa-jdbc-tool user add test1 –attribute=firstName=John –attribute=lastName=Doe

设置密码 ovirt-aaa-jdbc-tool user password-reset test1 –password-valid-to=“2025-08-01 12:00:00-0800”

查看用户信息 ovirt-aaa-jdbc-tool user show test1

编辑用户信息 ovirt-aaa-jdbc-tool user edit test1 –attribute=email=jdoe@example.com

删除用户 ovirt-aaa-jdbc-tool user delete test1

为内部管理员用户重设密码 ovirt-aaa-jdbc-tool user password-reset admin –password-valid-to=“2025-08-01 12:00:00Z”

禁用内部管理员用户,禁用默认的 admin 用户 ovirt-aaa-jdbc-tool user edit admin –flag=+disabled

要启用一个已禁用的用户，运行 ovirt-aaa-jdbc-tool user edit username –flag=-disabled

管理组： 创建一个新组 ovirt-aaa-jdbc-tool group add group1

为组添加用户。这些用户需要已存在。 ovirt-aaa-jdbc-tool group-manage useradd group1 –user=test1

查看组账户详情 ovirt-aaa-jdbc-tool group show group1

列出所有用户或组账户的信息 ovirt-aaa-jdbc-tool query –what=user ovirt-aaa-jdbc-tool query –what=group

列出名字以字母 j 开头的用户账户信息 ovirt-aaa-jdbc-tool query –what=user –pattern=“name=j*”

列出部门属性被设置为 marketing 的组的信息 ovirt-aaa-jdbc-tool query –what=group –pattern=“department=marketing”

运行以下命令显示所有设置 ovirt-aaa-jdbc-tool setting show

以下命令把所有用户账户的默认登录会话时间改为 60 分钟。它的默认值时 10080 分钟 ovirt-aaa-jdbc-tool setting set –name=MAX_LOGIN_MINUTES –value=60

以下命令更新了把用户账户被锁定前可以进行登录操作的失败次数。它的默认值是 5 ovirt-aaa-jdbc-tool setting set –name=MAX_FAILURES_SINCE_SUCCESS –value=3
```

## 九、配置cinder

##### 1.配置cinder provider 

登录ovirt管理界面，点击Administration→providers，点击右上角的添加按钮，选择添加新的provider。 在名称栏填入ovirt-cinder(可修改), 
类型选择：Openstack Block Storage， 
供应商URL填写：[http://http://192.168.21.100:8776](http://http//192.168.21.100:8776) (此处为openstack cinder api服务IP地址)。 
用户名:cinder， 
密码：xxxxxxxx, 此处为cinder用户的密码，此密码从/etc/kolla/passwords.yml 文件中的cinder_keystone_password 获取。 
租户名：service , 
验证名： <http://192.168.21.100:5000/v2.0> ，此处为openstackkeystone服务的认证URL。 
编辑完成以后点击测试，当界面弹出测试成功，访问了供应商。则表示配置成功。 

2.配置验证密钥 配置完成provider常规配置以后，还需要配置provider的验证密钥。 点击当前界面的验证密钥选项的右上角新建按钮。 在弹出的规划框界面，UUID 填入cinder的UUID。此UUID从openstack服务器上的cinder.conf获取。 

rbd_user = cinder 
rbd_secret_uuid = UUID 
3.值：填入此UUID对应的值。此值从ceph获取。获取方法：
ceph auth get client.cinder 
[client.cinder] 

```
      key = AQAZjXBZclIhAhAAH41aSF0GjlXEA4LBHYTEOg== 
```

此处的key 就是我们需要的值。
填写完成以后点击确定按钮。 
以上就是cinder provider的配置方法。

FAQ:rbd无法map(rbd feature disable) 参考解决方法：[http://www.zphj1987.com/2016/06/07/rbd%E6%97%A0%E6%B3%95map-rbd-feature-disable/](http://www.zphj1987.com/2016/06/07/rbd无法map-rbd-feature-disable/)

## 十、backup && restore

### environment

ovirt version: 4.2.2 

在互联网系统中，由于数据量很大，数据对企业来说就是企业的生命，因此数据的安全性就是企业首先要考虑的，解决数据安全性的首要措施就是对数据进行备份，对于ovirt engine来说可以对ovirt整个服务器采取备份，此种备份方式安全性最高，但是耗费的资源也比较大。另外一种解决方式就是对ovirt关键的文件进行备份，比如ovirt数据库和配置文件等，接下来我们就介绍ovirt对关键数据和配置文件进行备份。

### Backup

ovirt 提供专门的备份命令engine-backup 供管理员使用，具体的参数信息，用户可以使用如下命令查看： engine-backup –help 备份示例如下：

```
 engine-backup --mode=backup --file=backup1 --log=backup1.log 
```

\#mode:表示使用的模式，这里有两个参数选择：backup（备份），restore（还原） 
\#file:如果mode=backup ，则此处的file是指生成的备份文件。如果mode=restore，则此处表示已经存在的供还原的文件。 
\#log:备份过程中生成的日志文件。
执行备份命令以后在当前目录会生成一个名为backup1 的备份文件。

### Restore

前提条件： 
\1. 一台干净的服务器; 
\2. 安装ovirt-engine 但是没有set-up 
\3. 主机的hostname必须与备份主机的hostname保持一致 

还原过程：
\1. 执行 Restore 
\2. 执行 engine-setup 
我们在一台新安装的服务器上还原刚刚备份的backup1，使用示例如下： 
首先设置新服务器的网络，hostname等信息。将备份主机上生成的backup1文件和/opt/setup/answer 文件一起拷贝到新的主机。

```
 yum install ovirt-engine 
 engine-backup --mode=restore --log=restore1.log --file=backup1 --provision-db --provision-dwh-db --no-restore-permissions 
engine-setup --config-append=./answer --offline
```

执行玩上述三条命令以后，我们可以通过网页访问还原的主机https:*{IP} 进入web界面我们可以看到此时还原的主机与之前备份的主机信息完全一致。 参考文档：https://ovirt.org/develop/release-management/features/integration/engine-backup/*

## 十一、QOS

ovirt 中的 qos 包括存储、VM网络、主机网络、CPU，通过配置 qos 可以对可访问的资源进行精细的控制，创建 qos 的位置是:

```
Compute --> 数据中心 --> 选择一个数据中心 --> qos
```

##### 存储

存储服务质量定义了在一个存储域中的虚拟磁盘的最大吞吐级别和输入、输出操作的最大级别。为虚拟磁盘分配存储服务质量允许您对存储域的性能进行细化配置，并可以防止发生因为对一个虚拟磁盘的操作而影响到同一个存储域中的其它虚拟磁盘容量的问题。

##### 创建 qos

- 吞吐量：每秒数据读写量
- iops：每秒读写次数

使用 qos

在创建磁盘的时候需要选择一个磁盘配置集，而磁盘配置集可以设置 qos 属性，磁盘配置集的创建以及 qos 的创建位置是:

```
Storage --> Domains --> 选择一个数据存储域 --> 磁盘配置集 
```

创建一个配置集

##### VM网络

通过设置虚拟机网络服务质量，用户可以使用一个配置集来控制独立虚拟网络控制器的网络流量。它可以控制不同网络层上的带宽，并控制网络资源使用的情况。

##### 创建 qos

- 转入的：这个设置对流入网络的流量进行控制。选择/取消转入的选项来启用/禁用这个设置。
  - 平均值：流入网络流量的平均速度。
  - 高峰：流入网络流量的峰值速度。
  - Burst：burst 发生时的流入网络流量的速度。
- 转出的：这个设置对流出网络的流量进行控制。选择/取消转出的选项来启用/禁用这个设置。
  - 平均值：流入网络流量的平均速度。
  - 高峰：流入网络流量的峰值速度。
  - Burst：burst 发生时的流入网络流量的速度。

##### 使用 qos

创建虚拟机的时候需要选择一个 vnic 配置集，而 vnic 配置集可以设置 qos 属性，创建 vnic 配置集的位置是:

```
Network --> vNIC Profiles
```

##### 主机网络

主机网络服务器质量配置主机上的网络来控制通过物理接口的网络流量。使用主机网络服务质量可以通过控制在相同的物理网络接口控制器中的网络资源使用情况来对网络性能进行微调。这可以防止，因为物理网络上的一个网络有太多的网络数据造成同一个物理网络上的其它网络无法正常工作的情况出现。通过配置主机网络服务质量，可以使在同一个物理网络上的不同网络都正常工作。

创建 qos

- 转出的：应用到转出的网络流量的设置。
  - 加权重的共享：指定一个特定网络应该被分配的逻辑连接的容量（相对于连接到同一个逻辑连接的其它网络）。准确的共享取决于在这个连接上的所有网络共享的总和。在默认情况下，它的值在 1 到 100 之间。
  - 速率限制 [Mbps]：一个网络可以使用的最大带宽。
  - 实现的速率 [Mbps]：一个网络所需的最小带宽。实现的速率并不一定可以被保证，它取决于网络的架构以及同一个逻辑连接上的其它网络的“实现的速率”设置。

##### 使用 qos

主机网络的 qos 的使用是在创建网络的属性中进行设置的，具体的位置是:

```
Network --> 新建
```

CPU

CPU 服务质量定义了一个集群中的虚拟机可以从运行它的主机上获得的最大计算处理能力（以所占主机的所有计算处理能力的百分比表示）。通过为一个虚拟机关联一个 CPU 服务质量，可以防止因为集群中的一个虚拟机的负载占用太多计算处理资源而影响到同一集群中其它虚拟机可以使用的资源。

##### 创建 qos

- 限制：指所允许的最大处理能力（以百分比的形式，但不要包括 % 符号）

##### 使用 qos

在创建虚拟机或者编辑虚拟机的资源分配一项中可以设置 cpu 配置集，而 cpu 配置集可以设置 cpu qos 属性，创建 cpu 配置集的位置是:

```
Compute --> Cluster --> 选择一个 cluster --> CPU 配置集
```

新建一个配置集

## 十二、使用nat网络

/etc/libvirt/qemu/networks/

##### 1、创建NAT网络配置文件/etc/libvirt/qemu/networks/nat.xml,内容如下

```
<network>
        <name>nat</name>
        <uuid>b09d09a8-ebbd-476d-9045-e66012c9e83d</uuid>
        <forward mode='nat'/>
        <bridge name='natbr0' stp='on' delay='0' />
        <mac address='52:54:00:9D:82:DE'/>
        <ip address='192.168.1.1' netmask='255.255.255.0'>
            <dhcp>
                <range start='192.168.1.2' end='192.168.1.250' />
            </dhcp>
        </ip>
    </network>
```

##### 2、通过libvirt/virsh创建NAT网络

```
[root@ovirthost01 ~]# cat /etc/pki/vdsm/keys/libvirt_password 
    shibboleth
```

    [root@ovirthost01 ~]# virsh
    
    Welcome to virsh, the virtualization interactive terminal.
    
    Type:  'help' for help with commands
           'quit' to quit
    
    virsh # connect qemu:///system
    Please enter your authentication name: vdsm@ovirt
    Please enter your password: shibboleth
    
    virsh # net-list
     Name                 State      Autostart     Persistent
    ----------------------------------------------------------
     ;vdsmdummy;          active     no            no
     vdsm-ovirtmgmt       active     yes           yes
    
    virsh # net-define /etc/libvirt/qemu/networks/nat.xml
    Network nat defined from /etc/libvirt/qemu/networks/nat.xml
    
    virsh # net-autostart nat
    Network nat marked as autostarted
    
    virsh # net-start nat
    Network nat started
    
    virsh # net-list --all
     Name                 State      Autostart     Persistent
    ----------------------------------------------------------
     ;vdsmdummy;          active     no            no
     nat                  active     yes           yes
     vdsm-ovirtmgmt       active     yes           yes
    
    以上操作将创建nat功能的网桥，如下
    [root@ovirthost01 ~]# brctl show
    bridge name     bridge id               STP enabled     interfaces
    ;vdsmdummy;             8000.000000000000       no
    natbr0          8000.5254009d82de       yes             natbr0-nic
    ovirtmgmt               8000.b083fea27fed       no              p4p1
##### 3、安装vdsm-hook-extnet

```
[root@ovirthost01 ~]# yum install -y vdsm-hook-extnet
```

>  注：此处将下载extnet的hooks文件并存放到以下两目录

    [root@ovirthost01 ~]# ll /usr/libexec/vdsm/hooks/before_device_create
    total 4
    -rwxr-xr-x. 1 root root 1925 Jun  5 01:47 50_extnet
    [root@ovirthost01 ~]# ll /usr/libexec/vdsm/hooks/before_nic_hotplug
    total 4
    -rwxr-xr-x. 1 root root 1925 Jun  5 01:47 50_extnet
##### 4、添加自定义设备属性extnet

```
[root@ovirthost01 ~]# engine-config -s CustomDeviceProperties='{type=interface;prop={extnet=^[a-zA-Z0-9_ ---]+$}}'
    Please select a version:
    1. 3.0
    2. 3.1
    3. 3.2
    4. 3.3
    5. 3.4
    6. 3.5
    6
```

    [root@ovirthost01 ~]# engine-config -g CustomDeviceProperties
    CustomDeviceProperties:  version: 3.0
    CustomDeviceProperties:  version: 3.1
    CustomDeviceProperties:  version: 3.2
    CustomDeviceProperties:  version: 3.3
    CustomDeviceProperties: {type=interface;prop={SecurityGroups=^(?:(?:[0-9a-fA-F]{8}-(?:[0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}, *)*[0-9a-fA-F]{8}-(?:[0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}|)$}} version: 3.4
    CustomDeviceProperties: {type=interface;prop={extnet=^[a-zA-Z0-9_ ---]+$}} version: 3.5
    
    [root@ovirthost01 ~]# systemctl restart ovirt-engine
## 十三、ovirt-shell

https://access.redhat.com/documentation/en-us/red_hat_virtualization/4.1/html/rhevm_shell_guide/chap-using_the_cli

https://www.ovirt.org/develop/release-management/features/infra/cli/

```
yum -y install ovirt-engine-cli
pip install ovirt-shell

vim .ovirtshellrc

[cli]
autoconnect = True
autopage = True
[ovirt-shell]
username = admin@internal
renew_session = False
timeout = -1
extended_prompt = False
url = https://192.168.21.12/ovirt-engine/api
insecure = True
kerberos = False
filter = False
session_timeout = None
ca_file = /etc/pki/ovirt-engine/ca.pem
dont_validate_cert_chain = False
key_file = None
password = qwe
cert_file = None
```

```
ovirt-shell -E 'help'

连接
ovirt-shell -c -l "https://192.168.21.12/ovirt-engine/api" -P 443 -u admin@internal -I

列出资源
list vms
list vms --show-all

使用过滤查询列出资源
list vms --query "name=centos*"
list vms --kwargs "memory=1073741824"

列出子资源
list disks --vm-identifier nfs_desktop
list nics --vm-identifier demo	
list disks --vm-identifier nfs_desktop --kwargs "name=Disk 3"
list vms --kwargs "usb-enabled=True"

查看资源详细信息
show vm centos7
show vm --name nfs_desktop
show vm --id f4a51ae1-4f31-45ee-ab6d-d5965e3bcf71

查看子资源详细信息
show nic nic1 --vm-identifier demo

添加资源
add vm --name demo2 --template-name iscsi_desktop_tmpl --cluster-name Default_iscsi
add datacenter --name mydc --storage_type nfs --version-major 3 --version-minor 1

添加子资源
add nic --vm-identifier demo2 --network-name engine --name mynic

移除资源
remove vm aa

移除子资源
remove disk "Disk 1" --vm-identifier demo2

更新资源
update vm iscsi_desktop --description iscsi_desktop_desc
update vm iscsi_desktop --display-monitors 2 --description test1

更新子资源
update nic nic1 --vm-identifier demo --interface virtio

处理
action vm demo start --vm-display-type vnc --async true

移除子资源
action nic bond0 attach --host-identifier grey-vdsa

控制台
console 'my_vm'
console '7dff8517-7007-42cd-9cf7-b7a13a9d96b7'

action vm centos7 stop

创建数据中心
add datacenter --name test --comment "test" --description "test" --storage_type nfs --version-major 3 --version-minor 3

创建集群
add cluster --data_center-name test --name test --comment "test" 

添加主机到数据中心
list hosts --query "status=maintenance"
update host node-123 --cluster-name test

添加存储
# add storagedomain --name test --storage-type nfs --storage_format v3 --type data --host-name node-123 --storage-address 192.168.3.157 --storage-path /home/ovirt/nfs-test
add storagedomain --name test --datacenter-identifier test

添加ISO
add storagedomain --name iso --datacenter-identifier test

创建虚拟机
add vm --name test --template-name Blank --cluster-name test

给虚拟机新分配一个磁盘
add disk --name test_disk --size 10737418240 --interface virtio
list disks --query "alias=test_disk" | grep id
add disk --id 4357a838-2a99-4a05-a0b9-be6ee6ec5eaa --vm-identifier test

给虚拟机新分配一个网卡
add nic --vm-identifier test --name test --network-name ovirtmgmt

启动虚拟机
先列出 ISO 域名里可用的的镜像.
list files --storagedomain-identifier isoid 
给虚拟机增加一个 CDROM 设备.
add cdrom --vm-identifier test --file-id CentOS-6.5-x86_64-minimal.iso
启动虚拟机, 以 CDROM 作为启动设备
action vm test start --vm-stateless true --vm-os-boot "boot.dev=cdrom" --vm-display-ty

访问虚拟机
action vm test ticket --ticket-value "abc123"
show vm test | grep displaydisplay-address 
可以用 vnc 客户端打开 192.168.3.123:5900 加上上面设置的临时密码来访问该虚拟机了

脚本

    less /home/mpastern/script
    --------------------------
     
    list vms
    show vm test | grep status
    list vms --query "name=test*" --show-all | grep status
    list clusters
    list datacenters
    ...

执行脚本
From linux shell
     [mpastern@lp /]#  ovirt-shell -f /home/mpastern/script
From ovirt shell
    [oVirt shell (connected)]# file /home/mpastern/script
```

sdk

```
yum -y install gcc libxml2-devel python-devel
yum -y install ovirt-engine-cli


import logging
import ovirtsdk4 as sdk
import ovirtsdk4.types as types

logging.basicConfig(level=logging.DEBUG, filename='example.log')

connection = sdk.Connection(
    url='https://alpha/ovirt-engine/api',
    username='admin@internal',
    password='qwe',
    ca_file='/etc/pki/ovirt-engine/ca.pem',
    debug=True,
    log=logging.getLogger(),
)
vms_service = connection.system_service().vms_service()
vms = vms_service.list()
for vm in vms:
  print("%s: %s" % (vm.name, vm.id))
connection.close()

url="https://alpha/ovirt-engine/api"
user="admin@internal"
password="qwe"
vmid="5e2bf5a7-3e59-47d8-9f86-472a3d51a06e"

curl -X POST \
--insecure \
--header "Accept: application/xml" \
--header "Content-type: application/xml" \
--user "${user}:${password}" \
-d "<action></action>" \
"${url}/vms/${vmid}/shutdown"
```

rest api

https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_virtualization/3.6/html/rest_api_guide/

https://access.redhat.com/documentation/en-us/red_hat_virtualization/4.0/html-single/rest_api_guide/index

```
url="https://engine.example.com/ovirt-engine/api"
user="admin@internal"
password="..."
curl \
--verbose \
--cacert /etc/pki/ovirt-engine/ca.pem \
--user "${user}:${password}" \
--request POST \
--header "Version: 4" \
--header "Content-Type: application/xml" \
--header "Accept: application/xml" \
--data '
<vm>
  <name>myvm</name>
  <template>
    <name>Blank</name>
  </template>
  <cluster>
    <name>mycluster</name>
  </cluster>
</vm>
' \
"${url}/vms"

POST /ovirt-engine/api/vms/123/migrate

<action>
  <host id="2ab5e1da-b726-4274-bbf7-0a42b16a0fc3"/>
</action>

  <migration>
    <auto_converge>inherit</auto_converge>
    <bandwidth>
      <assignment_method>auto</assignment_method>
    </bandwidth>
    <compressed>inherit</compressed>
  </migration>
```



## 十四、其他问题

##### 1、The client is not authorized to request an authorization. It's required to access the system using FQDN.

```
/etc/ovirt-engine/engine.conf.d/11-setup-sso.conf

SSO_ALTERNATE_ENGINE_FQDNS

systemctl restart ovirt-engine.service
```

##### 2、跨域

```
$ ssh root@[ENGINE_FQDN] # log as `root` user in the oVirt engine machine
# engine-config -s CORSSupport=true # to turn on the CORS support for REST API
# engine-config -s CORSAllowDefaultOrigins=true # to allow CORS for all configured hosts
# systemctl restart ovirt-engine # to take effect
engine-config -s CORSAllowedOrigins=*
engine-config -s CORSSupport=true
engine-config -l |grep CORS
engine-config -g CORSAllowedOrigins
```

##### 3、迁移bug

>  迁移时，未完全开始就迁移，虚拟机状态可能会一直是migrate to，正常应该为Powering Up到Up

##### 4、v2v

```
yum -y install virt-v2v

virsh list --all
vdsm@ovirt
shibboleth

虚拟机需要关机

转换并导出到导出域（需要保证虚拟机是正常的）

virt-v2v -i libvirt -o rhev -os 192.168.21.12:/ovirt_export --network ovirtmgmt hl_v2v_test

在导入域中导入虚拟机

p2v
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/v2v_guide/chap-v2v_guide-p2v_migration_converting_physical_machines_to_virtual_machines
http://blog.csdn.net/lbwlh/article/details/51003595
```

##### 5、node 注册

```
vdsm-tool register --engine-fqdn IP_ADDRESS --check-fqdn false
```

##### 6、创建用户

```
ovirt-aaa-jdbc-tool user add hl
ovirt-aaa-jdbc-tool user password-reset hl --password-valid-to="2025-01-01 00:00:00-0800"
```

## 十五、ovirt virsh

```
vdsm@ovirt #yes
shibboleth
```

