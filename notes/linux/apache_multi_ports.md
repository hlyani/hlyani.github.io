# apache 多端口

##### 1、新建配置文件
```
touch /etc/apache2/sites-available/packages.conf
```
##### 2、编辑配置文件(/etc/apache2/sites-available/packages.conf)
``` /etc/apache2/sites-available/packages.conf
Listen 81

<VirtualHost *:81>
    ServerAdmin webmaster@localhost

    DocumentRoot /mnt/ramdisk
    <Directory />
        Options FollowSymLinks
        AllowOverride None
    </Directory>
    
    ErrorLog /var/log/apache2/error.log
    
    # Possible values include: debug, info, notice, warn, error, crit,
    # alert, emerg.
    LogLevel warn
    
    CustomLog /var/log/apache2/access.log combined

</VirtualHost>
```
##### 3、链接
```
ln -s /etc/apache2/sites-available/packages.conf /etc/apache2/sites-enabled
```
##### 4、在apache配置文件中（/etc/apache2/apache2.conf），添加如下内容：
``` /etc/apache2/apache2.conf
<Directory /mnt>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```
##### 4、重启apache服务
```
service apache2 restart
```