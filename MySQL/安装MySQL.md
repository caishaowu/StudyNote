### 先执行 rpm -qa | grep -i mysql  查看是否已安装了MySQL

CENTOS 执行RPM -IVH MYSQL-CLIENT-5.5.48-1.LINUX2.6.I386.RPM出现以下错误：</BR>
<B>WARNING: MYSQL-SERVER-5.5.48-1.LINUX2.6.I386.RPM: HEADER V3 DSA/SHA1 SIGNATURE, KEY ID 5072E1F5: NOKEY
error: Failed dependencies:
	libaio.so.1 is needed by MySQL-server-5.5.48-1.linux2.6.i386 </b>

出现原因：yum安装了旧版本的GPG keys造成的

解决：在原先命令的基础上加上 --force --nodeps即可

### 验证MySQL是否安装成功：


- cat /etc/passwd|grep mysql<br>


- cat /etc/group|grep mysql <br>


- mysqladmin --version

### 在启动MysQL服务出现以下错误：

Starting MySQL. ERROR! The server quit without updating PID file (/var/lib/mysql/centos.pid).
cat /var/lib/mysql/centos.err  查看错误<br>
原因如下：<br>
error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory<br>
解决方法如下：
**yum install -y libaio.so.1	安装libaio.so.1**<br>
**mysql_install_db --user=mysql --ldata=/var/lib/mysql/**      初始化 <br>

再次执行 service mysql start 成功！

### 设置root密码

/usr/bin/mysqladmin -u root password 123456

