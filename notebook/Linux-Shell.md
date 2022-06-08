<<<<<<< HEAD
## Shell:Openssh升级 ##

```shell

#!/bin/bash
#
#**************************************************************
#version          2
#Date:            2021-10-13
#filename：       install_opensssh.sh
#Description：    update openssh
#Author:          Nanhua Yang | Wulve Ou
#System:          CentOS 7
#**************************************************************
echo "输入格式:  install_openssh.sh [包路径]"
#获取输入路径,如果是相对路径转为绝对路径
input_path="$1"
if [ ${input_path:0:1} = "/"  ]; then
    pakge_path=$input_path
else
    pakge_path="$(pwd)/${input_path##*/}"
fi
#判断文件是否存在
if [ -f "$pakge_path" ]; then
    echo "$pakge_path 文件存在"
else 
    echo "$pakge_path 文件不存在"
    exit
fi

#输出输入路径
echo "输入的路径:   $input_path"
pakge_home="${pakge_path%/*}"
echo "上一级路径:   $pakge_home"
echo "包绝对路径:   $pakge_path"
pakge_name="${pakge_path##*/}"
echo "包名:         $pakge_name"
pakge_tarpath="${pakge_path%.tar.gz*}"
echo "解压文件路径: $pakge_tarpath"
pakge_tarname="${pakge_name%.tar.gz*}"
echo "解压文件名:   $pakge_tarname"

#安装依赖
yum install -y gcc gcc-c++ glibc make autoconf  openssl-devel pcre-devel  pam-devel
#cp /usr/lib/systemd/system/sshd.service /usr/lib/systemd/system/sshd.service.bak
#cp /usr/sbin/sshd /etc/sysconfig

#设置错误退出，如果有错误则退出
set -e

#删除旧版openssh
#rpm -qa |grep openssh
cp /etc/pam.d/sshd /etc/pam.d/sshd.bak
rpm -qa | grep openssh |xargs -n1  rpm -e --nodeps|| echo '继续执行'

#安装openssh
cd $pakge_home
tar -zvxf $pakge_path

#判断sshd.init文件是否存在
if [ -f "${pakge_tarpath}/contrib/redhat/sshd.init" ]; then
    echo "文件解压成功"
else 
    echo "$pakge_name 文件解压失败"
    exit
fi

cd $pakge_tarname||!echo '执行错误，请检查解压文件夹是否有问题。'||exit

install  -v -m700 -d /var/lib/sshd &&
chown    -v root:sys /var/lib/sshd &&
groupadd -g 50 sshd        &&
useradd  -c 'sshd PrivSep' \
         -d /var/lib/sshd  \
         -g sshd           \
         -s /bin/false     \
         -u 50 sshd

#./configure --prefix=/usr --sysconfdir=/etc/ssh --with-ssl-dir=/usr/local/openssl --with-md5-passwords --with-privsep-path=/var/lib/sshd &&
./configure --prefix=/usr         \
--sysconfdir=/etc/ssh             \
--with-ssl-dir=/usr/local/openssl \
--with-md5-passwords --with-zlib  \
--with-pam                        \
--with-tcp-wrappers               \
--with-zlib=/usr/local/lib64      \
--without-hardening               \
--with-privsep-path=/var/lib/sshd
#--with-libedit
#--with-kerberos5=/usr &&
make && 
chmod 600 /etc/ssh/ssh_host_rsa_key &&
chmod 600 /etc/ssh/ssh_host_ecdsa_key &&
chmod 600 /etc/ssh/ssh_host_ed25519_key &&
make install &&
install -v -m755    contrib/ssh-copy-id /usr/bin     &&
install -v -m644    contrib/ssh-copy-id.1 \
                    /usr/share/man/man1              &&
install -v -m755 -d /usr/share/doc/openssh-8.6p1     &&
install -v -m644    INSTALL LICENCE OVERVIEW README* \
                    /usr/share/doc/openssh-8.6p1

rm -rf /etc/pam.d/sshd
mv /etc/pam.d/sshd.bak /etc/pam.d/sshd
sed -i 's/#UsePAM no/UsePAM yes/g' /etc/ssh/sshd_config
sed -i 's/#Port 22/Port 22/g' /etc/ssh/sshd_config
sed -i 's/#PermitRootLogin..*/PermitRootLogin yes/g' /etc/ssh/sshd_config
sed -i 's/#ListenAddress 0.0.0.0/ListenAddress 0.0.0.0/g' /etc/ssh/sshd_config
sed -i 's/#ListenAddress ::/ListenAddress ::/g' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/g' /etc/ssh/sshd_config
sed -i 's/#PermitEmptyPasswords no/PermitEmptyPasswords no/g' /etc/ssh/sshd_config
cp ${pakge_tarpath}/contrib/redhat/sshd.init /etc/rc.d/init.d/sshd
chown root.root /etc/rc.d/init.d/sshd
chkconfig --add sshd
chkconfig sshd on
systemctl daemon-reload
systemctl start sshd.service

#输出相关变量数据
echo "------------------------------------------"
echo "输入的路径:   $input_path"
echo "上一级路径:   $pakge_home"
echo "包绝对路径:   $pakge_path"
echo "包名:         $pakge_name"
echo "解压文件路径: $pakge_tarpath"
echo "解压文件名:   $pakge_tarname"
echo "------------------------------------------"
```




## Shell:Install mysql ##

```shell
#!/bin/bash
set -e #错误退出
clear
echo "--------------------------------------------------------------
Shell:        $0                  
Version:      1                   
Data:         2021-11-09          
Description:  Use yum Install Mysql. 
Author:       Wulve               
System:       CentOS 7            
Instructions: $0 [-ins/-reins/-unins] [-mysql_rpm_path]
--------------------------------------------------------------"
mysql_rpm_path_57=http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
mysql_rpm_path_80=https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
case $1 in
    -ins)
        echo "Mysql is Installing! "
    ;;
    -reins)
        systemctl stop mysqld|| ! echo 'Mysql服务未运行' 
        rm -rf /var/log/mysqld.log || ! echo '/var/log/mysqld.log不存在'
        rpm -e --nodeps $(rpm -qa | grep mysql)|| ! echo 'RPM已删除'
        rm -rf $(find / -name mysql)|| ! echo 'Mysql文件不存在'
    ;;
    -unins)
        systemctl stop mysqld
        rpm -e --nodeps $(rpm -qa | grep mysql)
        rm -rf $(find / -name mysql)
        exit
    ;;
    *)
    ;;
esac

mysql_rpm_path=$2
#如果地址输入为空，使用默认配置
if  [ ! -n "$mysql_rpm_path" ] ;then
    echo "The mysql_rpm_path is null, that will be default configuration."
    read -r -p "Please input the Mysql Version(5.7 or 8.0):" input_version
    case $input_version in
    5.7)
        echo "The mysql_rpm_path will be: $mysql_rpm_path_57"
        mysql_rpm_path="$mysql_rpm_path_57"
    ;;
    8.0)
        echo "The mysql_rpm_path will be: $mysql_rpm_path_80"
        mysql_rpm_path=$mysql_rpm_path_80
    ;;
    *)
        echo "The mysql_rpm_path will be: $mysql_rpm_path_80"
        mysql_rpm_path=$mysql_rpm_path_80
    ;;
    esac
fi
mysql_rpm="${mysql_rpm_path##*/}"
#判断是否已经有rpm包
echo "mysql_rpm=$mysql_rpm"
if [ -f "./$mysql_rpm" ]; then
    echo "The RPM exists."
else 
    wget $mysql_rpm_path
fi
echo "[Finish] 判断是否已经有rpm包"
#修改防火墙配置
change_iptables () {
cp /etc/sysconfig/iptables /etc/sysconfig/iptables.bak
sed -i 's/tcp --dport 22 -j ACCEPT/&\n-A INPUT -p tcp --dport 3306 -j ACCEPT/' /etc/sysconfig/iptables
service iptables restart
}|| ! echo '防火墙配置错误'
echo "[Finish] 修改防火墙配置"
#检测是否安装
systemctl start mysqld.service &>/dev/null|| ! echo 'Mysql服务未运行'
if [ $? -ne 0 ];then
    yum -y install $mysql_rpm
    yum -y install mysql-community-server
else
    echo "Mysql is installed."
    sleep 1
    systemctl start mysqld.service      #启动服务
    sleep 1
    systemctl status mysqld             #查看服务状态
    systemctl enable mysqld             #开机自启
    systemctl daemon-reload
    exit
fi
sleep 1
systemctl start mysqld.service      #启动服务
sleep 1
systemctl status mysqld             #查看服务状态
systemctl enable mysqld             #开机自启
systemctl daemon-reload
#1.新添加 unit 配置文件时需要执行 daemon-reload 子命令,
#2.有 unit 的配置文件发生变化时也需要执行 daemon-reload 子命令
echo "[Finish] 安装部署"
#修改密码
yum install -y expect
old_passwd=`grep "password" /var/log/mysqld.log |awk 'NR==1{print $NF}'`
new_passwd="Hello@2021"
expect << EOF
spawn mysql -uroot -p
expect ":" {send "$old_passwd\r"}
expect "> " {send "set global validate_password.policy=0;\r"}
expect "> " {send "set global validate_password.mixed_case_count=0;\r"}
expect "> " {send "set global validate_password.number_count=0;\r"}
expect "> " {send "set global validate_password.special_char_count=0;\r"}
expect "> " {send "set global validate_password.length=6;\r"}
expect "> " {send "alter user 'root'@'localhost' identified by '$new_passwd';\r"}
expect "#" {send "exit\r"}
EOF
echo "[Finish] 修改密码"
#修改防火墙配置
yum install -y iptables iptables-services
change_iptables
echo "[Finish] 修改防火墙配置"
#修改为UTF-8格式
sed -i 's/mysqld]/&\ncharacter_set_server=utf8/' /etc/my.cnf
sed -i "s/server=utf8/&\ninit_connect='SET NAMES utf8'/" /etc/my.cnf
systemctl restart mysqld
echo "[Finish] 修改为UTF-8格式"
#验证密码
expect << EOF
spawn mysql -uroot -p
expect ":" {send "$new_passwd\r"}
expect ">" {send "exit\r"}
EOF
echo "[Finish] 验证密码"
echo "Mysql setup finish and Password is '$new_passwd'"
#释放变量
unset old_passwd
unset new_passwd
```

## Shell:设置静态IP ##

```shell
#! /bin/bash
set -e #错误退出
clear
echo "**************************************************************"
echo "Shell:    $0               "
echo "版本:     1                "          
echo "日期:     2021-10-13       "
echo "描述:     用于静态IP的设置  "
echo "作者:     Wulve            "
echo "系统:     CentOS 7         "
echo "使用:     $0 [网卡名称]    "
echo "**************************************************************"
echo
echo "配置的网卡为: $1"
ifcfg_path=/etc/sysconfig/network-scripts/ifcfg-$1;
#恢复网络配置
if [ "$2" = "-r" ]; then
    echo "--------------------------------------------------------------"
    echo "准备恢复网络配置!"
    #判断网卡配置是否无误。
    if [ -f "$ifcfg_path.bak.ByWulve" ]; then
        cp $ifcfg_path.bak.ByWulve $ifcfg_path|| ! echo '恢复出错!' || exit
        echo "重启networkf服务"
        service network restart
        echo "重启networkf服务完成"
        echo "恢复完成!"
        echo "--------------------------------------------------------------"
        echo "当前网络配置信息:"
        ifconfig
        echo "--------------------------------------------------------------"
        exit
    else 
        echo "$1网卡配置备份文件不存在, 或者网卡有误。"
        ifconfig
        exit
    fi
fi
#判断网卡配置是否无误。
if [ -f "$ifcfg_path" ]; then
    echo "$1网卡配置存在"
else 
    echo "$1网卡配置不存在,请确认网卡名称。"
    ifconfig
    exit
fi
echo

#获取相关输入数据
echo "--------------------------------------------------------------"
echo "网卡配置文件: $ifcfg_path"
read -r -p "请输入静态IP    :" input_IPADDR
read -r -p "请输入子网掩码  :" input_NETMASK
if  [ ! -n "$input_NETMASK" ] ;then
    echo "子网掩码未配置,使用:255.255.255.0"
    input_NETMASK=255.255.255.0
fi
read -r -p "请输入网关地址  :" input_GATEWAY
if  [ ! -n "$input_DNS1" ] ;then
    echo "DNS未配置,使用:114.114.114.114"
    input_NETMASK=114.114.114.114
fi
read -r -p "请输入DNS地址   :" input_DNS1
if  [ ! -n "$input_DNS1" ] ;then
    echo "DNS未配置,使用:114.114.114.114"
    input_NETMASK=114.114.114.114
fi
#echo "--------------------------------------------------------------"
#测试使用
# input_IPADDR=192.168.129.128
# input_NETMASK=255.255.255.0
# input_GATEWAY=192.168.129.2
# input_DNS1=114.114.114.114
# echo "相关配置信息:"
# echo "静态IP    :$input_IPADDR" 
# echo "子网掩码  :$input_NETMASK" 
# echo "网关地址  :$input_GATEWAY" 
# echo "DNS地址   :$input_DNS1" 
#备份配置
echo "--------------------------------------------------------------"
echo "备份$1网卡配置"
if [ -f "$ifcfg_path.bak.ByWulve" ]; then
    echo "$1网卡备份配置已经存在, 本次不备份."
else 
    cp $ifcfg_path $ifcfg_path.bak.ByWulve
    echo "备份完成备份位置:$ifcfg_path.bak.ByWulve"
fi
read -r -p "配置错误将会断开SSH连接, 是否继续执行? [Y/N] " input
case $input in
    [yY][eE][sS]|[yY])
        echo "同意，继续执行"
        ;;
    [nN][oO]|[nN])
        echo "拒绝，执行退出"
        exit 1
        ;;
    *)
        echo "无效输入,执行退出"
        exit 1
        ;;
esac
#修改配置
echo "--------------------------------------------------------------"
echo "修改$1网卡配置"
sed -i "4c\BOOTPROTO="static" #修改为静态IP,默认为dhcp(已修改)" "$ifcfg_path"
sed -i "6c\IPV4_FAILURE_FATAL="yes" #开启IPV4(已修改)"         "$ifcfg_path" #开启IPV4
sed -i "7c\IPV6INIT="no" #关闭IPV6(已修改)"                    "$ifcfg_path" #关闭IPV6
sed -i "7c\#IPV6_ADDR_GEN_MODE="stable-privacy" #注释(已修改)" "$ifcfg_path" #注释第11行IPV6_ADDR_GEN_MODE="stable-privacy"
echo "IPADDR="$input_IPADDR" #设置的静态IP地址">>"$ifcfg_path"
echo "NETMASK="$input_NETMASK" #设置子网掩码">>"$ifcfg_path"
echo "GATEWAY="$input_GATEWAY" #设置网关地址">>"$ifcfg_path"
if  [ ! -n "$input_DNS1" ] ;then
    echo "未输入DNS地址，不配置"
else
    echo "DNS1="$input_DNS1" #设置DNS地址">>"$ifcfg_path"
fi
echo "$1网卡配置修改完成"
#重启network服务
echo "--------------------------------------------------------------"
echo "重启networkf服务"
service network restart
echo "重启networkf服务完成"
#ping www.baidu.com
echo "--------------------------------------------------------------"
echo "准备ping www.baidu.com 测试"
ping -c3 baidu.com
#输出相关配置
echo "--------------------------------------------------------------"
echo "相关配置信息:"
echo "静态IP    :$input_IPADDR" 
echo "子网掩码  :$input_NETMASK" 
echo "网关地址  :$input_GATEWAY" 
echo "DNS地址   :$input_DNS1" 
echo "静态配置完成, 如果使用有问题:"
echo "使用'$0 $1 -r'命令可以恢复初始网络配置"
echo "--------------------------------------------------------------"

```

## Shell:设置静态IP 2 ##

```shell

#! /bin/bash
set -e #错误退出
clear
echo "--------------------------------------------------------------
Shell:        $0                  
Version:      2                   
Data:         2021-10-13          
Description:  Settings static IP. 
Author:       Wulve               
System:       CentOS 7            
Instructions: $0 [Network card’s name]    
--------------------------------------------------------------
The network card being configured is: $1
正在配置的网卡是: $1"
ifcfg_path=/etc/sysconfig/network-scripts/ifcfg-$1;
#恢复网络配置
if [ "$2" = "-r" ]; then
    echo "--------------------------------------------------------------"
    echo "Prepare to restore network configuration"
    echo "准备恢复网络配置!"
    #判断网卡配置是否无误。
    if [ -f "$ifcfg_path.bak.ByWulve" ]; then
        cp $ifcfg_path.bak.ByWulve $ifcfg_path|| ! echo '恢复出错!' || exit
        echo "Restart networkf service"
        echo "重启networkf服务"
            service network restart
        echo "Restart the networkf service to complete！"
        echo "重启networkf服务完成"
        echo "Recovery is complete!"
        echo "恢复完成!"
        echo "--------------------------------------------------------------"
        echo "Current network configuration information:"
        echo "当前网络配置信息:"
            ifconfig
        echo "--------------------------------------------------------------"
        exit
    else 
        echo "$1 The network card configuration backup file does not exist, or the network card is incorrect."
        echo "$1网卡配置备份文件不存在, 或者网卡有误。"
        ifconfig
        exit
    fi
fi
#判断网卡配置是否无误。
if [ -f "$ifcfg_path" ]; then
    echo "$1 Network card configuration exists"
    echo "$1网卡配置存在"
else 
    echo "$1 The network card configuration does not exist, please confirm the network card name."
    echo "$1网卡配置不存在,请确认网卡名称。"
    ifconfig
    exit
fi
echo

#获取相关输入数据
echo "--------------------------------------------------------------"
echo "Network card configuration file location: $ifcfg_path"
echo "网卡配置文件位置: $ifcfg_path"
read -r -p "Please input the static IP (输入IP)    :" input_IPADDR             #设置静态IP
read -r -p "Please input the NETMASK (输入子网掩码):" input_NETMASK            #设置子网掩码
if  [ ! -n "$input_NETMASK" ] ;then
    echo "The NETMASK is null,that will be: 255.255.255.0"
    input_NETMASK=255.255.255.0
fi
read -r -p "Please input the GATEWAY (输入默认网关):" input_GATEWAY                #设置默认网关
read -r -p "Please input the DNS (输入DNS)         :" input_DNS1                       #设置DNS地址

#echo "--------------------------------------------------------------"
#测试使用
# input_IPADDR=192.168.129.128
# input_NETMASK=255.255.255.0
# input_GATEWAY=192.168.129.2
# input_DNS1=114.114.114.114
# echo "相关配置信息:"
# echo "静态IP    :$input_IPADDR" 
# echo "子网掩码  :$input_NETMASK" 
# echo "网关地址  :$input_GATEWAY" 
# echo "DNS地址   :$input_DNS1" 
#备份配置
echo "--------------------------------------------------------------"
echo "Backup $1 network card configuration
备份$1网卡配置"
if [ -f "$ifcfg_path.bak.ByWulve" ]; then
    echo "The backup configuration of the $1 network card already exists, and it will not be backed up this time."
    echo "$1网卡备份配置已经存在, 本次不备份."
else 
    cp $ifcfg_path $ifcfg_path.bak.ByWulve
    echo "The backup is complete, the backup location:$ifcfg_path.bak.ByWulve"
    echo "备份完成备份位置:$ifcfg_path.bak.ByWulve"
fi
echo "The configuration error will disconnect the SSH.Do you want to continue?[Y/N]"
read -r -p "配置错误将会断开SSH连接, 是否继续执行? [Y/N] " input
case $input in
    [yY][eE][sS]|[yY])
        echo "Agree, continue"
        echo "同意，继续执行"
        ;;
    [nN][oO]|[nN])
        echo "Reject, execute exit"
        echo "拒绝，执行退出"
        exit 1
        ;;
    *)
        echo "Invalid input, execute exit"
        echo "无效输入,执行退出"
        exit 1
        ;;
esac
#修改配置
echo "--------------------------------------------------------------"
echo "Modifying the configuration of the $1 network card
修改$1网卡配置"
sed -i 's/BOOTPROTO="dhcp"/BOOTPROTO="static"/g' "$ifcfg_path"
#sed -i "4c\BOOTPROTO="static" #修改为静态IP,默认为dhcp(已修改)" "$ifcfg_path"
sed -i 's/IPV4_FAILURE_FATAL="no"/IPV4_FAILURE_FATAL="yes"/g' "$ifcfg_path"
#sed -i "6c\IPV4_FAILURE_FATAL="yes" #开启IPV4(已修改)"         "$ifcfg_path" #开启IPV4
sed -i 's/IPV6INIT="yes"/IPV6INIT="no"/g' "$ifcfg_path"
#sed -i "7c\IPV6INIT="no" #关闭IPV6(已修改)"                    "$ifcfg_path" #关闭IPV6
sed -i 's/^IPV6_ADDR_GEN_MODE="stable-privacy"/#IPV6_ADDR_GEN_MODE="stable-privacy"/g' "$ifcfg_path"
#sed -i "7c\#IPV6_ADDR_GEN_MODE="stable-privacy" #注释(已修改)" "$ifcfg_path" #注释第11行IPV6_ADDR_GEN_MODE="stable-privacy"
echo " ">>"$ifcfg_path"
echo "IPADDR="$input_IPADDR" #设置的静态IP地址">>"$ifcfg_path"
echo "NETMASK="$input_NETMASK" #设置子网掩码">>"$ifcfg_path"
echo "GATEWAY="$input_GATEWAY" #设置网关地址">>"$ifcfg_path"
if  [ ! -n "$input_DNS1" ] ;then
    echo "DNS address is not entered, DNS is not configured"
    echo "未输入DNS地址，不配置DNS"
    input_DNS1="NULL"
else
    echo "DNS1="$input_DNS1" #设置DNS地址">>"$ifcfg_path"
fi
echo "$1 network card configuration has been modified."
echo "$1网卡配置修改完成"
#重启network服务
echo "--------------------------------------------------------------"
echo "Restart networkf service."
echo "重启networkf服务."
    service network restart
echo "Restart the network service to complete."
echo "重启networkf服务完成"
#ping www.baidu.com
echo "--------------------------------------------------------------"
echo "Prepare to ping www.baidu.com to test."
echo "准备ping www.baidu.com 测试"
ping -c3 baidu.com
#输出相关配置
echo "--------------------------------------------------------------
Related configuration information:
相关配置信息:
Static IP(静态IP) :$input_IPADDR
NETMASK(子网掩码) :$input_NETMASK
GATEWAY(网关地址) :$input_GATEWAY
DNS(DNS地址)      :$input_DNS1
Use the'$0 $1 -r' command to restore the initial network configuration
使用'$0 $1 -r'命令可以恢复初始网络配置
--------------------------------------------------------------"

```

## Shell:JDK配置 ##

```shell
#!/bin/bash
#set -e #错误退出
clear
echo "**************************************************************"
echo "Shell:    $0               "
echo "版本:     1                "          
echo "日期:     2021-10-13       "
echo "描述:     用于JDK的配置  "
echo "作者:     Wulve            "
echo "系统:     CentOS 7         "
echo "使用:     $0 [包路径]    "
echo "**************************************************************"
#获取输入路径,如果是相对路径转为绝对路径
input_path="$1"
if [ ${input_path:0:1} = "/"  ]; then
    pakge_path=$input_path
else
    pakge_path="$(pwd)/${input_path##*/}"
fi
#判断文件是否存在
if [ -f "$pakge_path" ]; then
    echo "$pakge_path 文件存在"
else 
    echo "$pakge_path 文件不存在"
    exit
fi
java -version
if [ $? -ne 0 ];then #如果上一个命令退出没有数据，则执行
    mkdir /usr/local/java
    cp $pakge_path /usr/local/java
    cd /usr/local/java && tar -xvf $pakge_path
    echo "export JAVA_HOME=/usr/local/java/jdk1.8.0_251" >> /etc/profile
    echo 'export JRE_HOME=$JAVA_HOME/jre' >> /etc/profile
    echo 'export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin' >> /etc/profile
    echo 'export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib' >> /etc/profile
    
    source /etc/profile
else
    echo "JDK is instelled."
    exit
fi
```

## Shell:CPU占用率高的进程 ##

```shell
#!/bin/bash
#topk 前k个进程
#samplingTime 采样时间, 单位秒
#interval 采样间隔时间, 单位秒

TOPK={{topk}}
SECS={{samplingTime}}
INTERVAL={{interval}}

STEPS=$(( $SECS / $INTERVAL ))
TEMP_FILE_PREFIX="/tmp/tat_public_cpu_usage"
echo Watching CPU usage...
for((i=0;i<$STEPS;i++))
do
  ps -eocomm,pcpu | tail -n +2 >> $TEMP_FILE_PREFIX.$$
  sleep $INTERVAL
done
echo
echo CPU eaters :
cat $TEMP_FILE_PREFIX.$$ | \
awk '
{ process[$1]+=$2;}
END{
  for(i in process) {
    printf("%-20s %s\n",i, process[i]) ;
  }
}' | sort -nrk 2 | head -n $TOPK
rm $TEMP_FILE_PREFIX.$$
```

## Shell:显示僵尸进程 ##

```shell
#!/bin/bash
processes=$(ps ax -o user,pid,ppid,pgid,args,stat,start,time)
zombies=$(echo -e "${processes}" | grep -E "\s(Z|z|Z.*)\s")
if [ $? == 1 ]; then
  echo "no zombie processes exists on machine"
else
  echo -e "${processes}" | head -1
  echo "$zombies"
fi
```

## Shell:清理磁盘 ##

```shell
#!/bin/bash
#批量在多台Linux实例上清理磁盘
#delayTime  7d  文件的有效时间。如 
#          7d（代表7天），1h（代表1小时），5m（代表5分钟），默认是7d
#filePath          清理文件路径。如：/root/log/
#matchPattern   清理文件匹配格式，如 *.log。 支持正则匹配

  
function deletefiles() {
  if [ ! -d $2 ]; then
    echo "The specified directory("$2") is not exist."
    return
  fi
  
  expiredTimeUnit=${1: -1}
  expiredTimeValue=${1:0:-1}
  
  if [ "$expiredTimeUnit" = "d" ]; then
    expiredTime=$(($expiredTimeValue * 24 * 60 * 60))
  elif [ "$expiredTimeUnit" = "h" ]; then
    expiredTime=$(($expiredTimeValue * 60 * 60))
  elif [ "$expiredTimeUnit" = "m" ]; then
    expiredTime=$(($expiredTimeValue * 60))
  else
    echo "The unit("$expiredTimeUnit") of file age is not supported."
    return
  fi
  
  for file in $(find $2 -type f -name "$3"); do
    local currentDate=$(date +%s)
    local modifyDate=$(stat -c %Y $file)
    local existTime=$(($currentDate - $modifyDate))
  
    if [ $existTime -gt $expiredTime ]; then
      echo "File cleaning succeeded,path:"$file"."
      rm -f $file
    fi
  done
}
  
deletefiles {{delayTime}} {{filePath}} "{{matchPattern}}"
```

## Shell:查看占用空间 ##

```shell
#!/bin/bash
#查看目录占用磁盘空间大小
#directory 目标目录

du -sh {{directory}}
```

## Shell:批量安装或卸载包 ##

```shell
#!/bin/bash
#批量在多台Linux实例上安装或卸载包。
#Installer  apt-get  
#     包管理器,只允许输入以下值[apt-get,yum]
#Action  install        
#     安装或卸载 package,只允许输入以下值[install,uninstall]
#packageName
#     yum/apt-get   安装的包名

function configurePackages() {
    installer=$1
    action=$2
    packageName=$3
    if [ "$installer" = "yum" ]; then
        if [ "$action" = "install" ]; then
            yum install -y $packageName
            if [ $? -ne 0 ]; then
                echo "Package install failed. Please check your command"
                exit 1
            fi
        elif [ "$action" = "uninstall" ]; then
            yum remove -y $packageName
            if [ $? -ne 0 ]; then
                echo "Package uninstall failed. Please check your command"
                exit 1
            fi
        else
            echo "Package command must be install or uninstall"
            exit 1
        fi
    elif [ "$installer" = "apt-get" ]; then
        if [ "$action" = "install" ]; then
            apt-get -y install $packageName
            if [ $? -ne 0 ]; then
                echo "Package install failed. Please check your command"
                exit 1
            fi
        elif [ "$action" = "uninstall" ]; then
            apt-get -y remove $packageName
            if [ $? -ne 0 ]; then
                echo "Package uninstall failed. Please check your command"
                exit 1
            fi
        else
            echo "Package command must be install or uninstall"
            exit 1
        fi
    else
        echo "Unknown package installer. Only support yum/apt-get"
        exit 1
    fi
}
  
configurePackages {{installer}} {{action}} {{packageName}}
```

=======
## Shell:Openssh升级 ##

```shell

#!/bin/bash
#
#**************************************************************
#version          2
#Date:            2021-10-13
#filename：       install_opensssh.sh
#Description：    update openssh
#Author:          Nanhua Yang | Wulve Ou
#System:          CentOS 7
#**************************************************************
echo "输入格式:  install_openssh.sh [包路径]"
#获取输入路径,如果是相对路径转为绝对路径
input_path="$1"
if [ ${input_path:0:1} = "/"  ]; then
    pakge_path=$input_path
else
    pakge_path="$(pwd)/${input_path##*/}"
fi
#判断文件是否存在
if [ -f "$pakge_path" ]; then
    echo "$pakge_path 文件存在"
else 
    echo "$pakge_path 文件不存在"
    exit
fi

#输出输入路径
echo "输入的路径:   $input_path"
pakge_home="${pakge_path%/*}"
echo "上一级路径:   $pakge_home"
echo "包绝对路径:   $pakge_path"
pakge_name="${pakge_path##*/}"
echo "包名:         $pakge_name"
pakge_tarpath="${pakge_path%.tar.gz*}"
echo "解压文件路径: $pakge_tarpath"
pakge_tarname="${pakge_name%.tar.gz*}"
echo "解压文件名:   $pakge_tarname"

#安装依赖
yum install -y gcc gcc-c++ glibc make autoconf  openssl-devel pcre-devel  pam-devel
#cp /usr/lib/systemd/system/sshd.service /usr/lib/systemd/system/sshd.service.bak
#cp /usr/sbin/sshd /etc/sysconfig

#设置错误退出，如果有错误则退出
set -e

#删除旧版openssh
#rpm -qa |grep openssh
cp /etc/pam.d/sshd /etc/pam.d/sshd.bak
rpm -qa | grep openssh |xargs -n1  rpm -e --nodeps|| echo '继续执行'

#安装openssh
cd $pakge_home
tar -zvxf $pakge_path

#判断sshd.init文件是否存在
if [ -f "${pakge_tarpath}/contrib/redhat/sshd.init" ]; then
    echo "文件解压成功"
else 
    echo "$pakge_name 文件解压失败"
    exit
fi

cd $pakge_tarname||!echo '执行错误，请检查解压文件夹是否有问题。'||exit

install  -v -m700 -d /var/lib/sshd &&
chown    -v root:sys /var/lib/sshd &&
groupadd -g 50 sshd        &&
useradd  -c 'sshd PrivSep' \
         -d /var/lib/sshd  \
         -g sshd           \
         -s /bin/false     \
         -u 50 sshd

#./configure --prefix=/usr --sysconfdir=/etc/ssh --with-ssl-dir=/usr/local/openssl --with-md5-passwords --with-privsep-path=/var/lib/sshd &&
./configure --prefix=/usr         \
--sysconfdir=/etc/ssh             \
--with-ssl-dir=/usr/local/openssl \
--with-md5-passwords --with-zlib  \
--with-pam                        \
--with-tcp-wrappers               \
--with-zlib=/usr/local/lib64      \
--without-hardening               \
--with-privsep-path=/var/lib/sshd
#--with-libedit
#--with-kerberos5=/usr &&
make && 
chmod 600 /etc/ssh/ssh_host_rsa_key &&
chmod 600 /etc/ssh/ssh_host_ecdsa_key &&
chmod 600 /etc/ssh/ssh_host_ed25519_key &&
make install &&
install -v -m755    contrib/ssh-copy-id /usr/bin     &&
install -v -m644    contrib/ssh-copy-id.1 \
                    /usr/share/man/man1              &&
install -v -m755 -d /usr/share/doc/openssh-8.6p1     &&
install -v -m644    INSTALL LICENCE OVERVIEW README* \
                    /usr/share/doc/openssh-8.6p1

rm -rf /etc/pam.d/sshd
mv /etc/pam.d/sshd.bak /etc/pam.d/sshd
sed -i 's/#UsePAM no/UsePAM yes/g' /etc/ssh/sshd_config
sed -i 's/#Port 22/Port 22/g' /etc/ssh/sshd_config
sed -i 's/#PermitRootLogin..*/PermitRootLogin yes/g' /etc/ssh/sshd_config
sed -i 's/#ListenAddress 0.0.0.0/ListenAddress 0.0.0.0/g' /etc/ssh/sshd_config
sed -i 's/#ListenAddress ::/ListenAddress ::/g' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/g' /etc/ssh/sshd_config
sed -i 's/#PermitEmptyPasswords no/PermitEmptyPasswords no/g' /etc/ssh/sshd_config
cp ${pakge_tarpath}/contrib/redhat/sshd.init /etc/rc.d/init.d/sshd
chown root.root /etc/rc.d/init.d/sshd
chkconfig --add sshd
chkconfig sshd on
systemctl daemon-reload
systemctl start sshd.service

#输出相关变量数据
echo "------------------------------------------"
echo "输入的路径:   $input_path"
echo "上一级路径:   $pakge_home"
echo "包绝对路径:   $pakge_path"
echo "包名:         $pakge_name"
echo "解压文件路径: $pakge_tarpath"
echo "解压文件名:   $pakge_tarname"
echo "------------------------------------------"
```




## Shell:Install mysql ##

```shell
#!/bin/bash
set -e #错误退出
clear
echo "--------------------------------------------------------------
Shell:        $0                  
Version:      1                   
Data:         2021-11-09          
Description:  Use yum Install Mysql. 
Author:       Wulve               
System:       CentOS 7            
Instructions: $0 [-ins/-reins/-unins] [-mysql_rpm_path]
--------------------------------------------------------------"
mysql_rpm_path_57=http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
mysql_rpm_path_80=https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
case $1 in
    -ins)
        echo "Mysql is Installing! "
    ;;
    -reins)
        systemctl stop mysqld|| ! echo 'Mysql服务未运行' 
        rm -rf /var/log/mysqld.log || ! echo '/var/log/mysqld.log不存在'
        rpm -e --nodeps $(rpm -qa | grep mysql)|| ! echo 'RPM已删除'
        rm -rf $(find / -name mysql)|| ! echo 'Mysql文件不存在'
    ;;
    -unins)
        systemctl stop mysqld
        rpm -e --nodeps $(rpm -qa | grep mysql)
        rm -rf $(find / -name mysql)
        exit
    ;;
    *)
    ;;
esac

mysql_rpm_path=$2
#如果地址输入为空，使用默认配置
if  [ ! -n "$mysql_rpm_path" ] ;then
    echo "The mysql_rpm_path is null, that will be default configuration."
    read -r -p "Please input the Mysql Version(5.7 or 8.0):" input_version
    case $input_version in
    5.7)
        echo "The mysql_rpm_path will be: $mysql_rpm_path_57"
        mysql_rpm_path="$mysql_rpm_path_57"
    ;;
    8.0)
        echo "The mysql_rpm_path will be: $mysql_rpm_path_80"
        mysql_rpm_path=$mysql_rpm_path_80
    ;;
    *)
        echo "The mysql_rpm_path will be: $mysql_rpm_path_80"
        mysql_rpm_path=$mysql_rpm_path_80
    ;;
    esac
fi
mysql_rpm="${mysql_rpm_path##*/}"
#判断是否已经有rpm包
echo "mysql_rpm=$mysql_rpm"
if [ -f "./$mysql_rpm" ]; then
    echo "The RPM exists."
else 
    wget $mysql_rpm_path
fi
echo "[Finish] 判断是否已经有rpm包"
#修改防火墙配置
change_iptables () {
cp /etc/sysconfig/iptables /etc/sysconfig/iptables.bak
sed -i 's/tcp --dport 22 -j ACCEPT/&\n-A INPUT -p tcp --dport 3306 -j ACCEPT/' /etc/sysconfig/iptables
service iptables restart
}|| ! echo '防火墙配置错误'
echo "[Finish] 修改防火墙配置"
#检测是否安装
systemctl start mysqld.service &>/dev/null|| ! echo 'Mysql服务未运行'
if [ $? -ne 0 ];then
    yum -y install $mysql_rpm
    yum -y install mysql-community-server
else
    echo "Mysql is installed."
    sleep 1
    systemctl start mysqld.service      #启动服务
    sleep 1
    systemctl status mysqld             #查看服务状态
    systemctl enable mysqld             #开机自启
    systemctl daemon-reload
    exit
fi
sleep 1
systemctl start mysqld.service      #启动服务
sleep 1
systemctl status mysqld             #查看服务状态
systemctl enable mysqld             #开机自启
systemctl daemon-reload
#1.新添加 unit 配置文件时需要执行 daemon-reload 子命令,
#2.有 unit 的配置文件发生变化时也需要执行 daemon-reload 子命令
echo "[Finish] 安装部署"
#修改密码
yum install -y expect
old_passwd=`grep "password" /var/log/mysqld.log |awk 'NR==1{print $NF}'`
new_passwd="Hello@2021"
expect << EOF
spawn mysql -uroot -p
expect ":" {send "$old_passwd\r"}
expect "> " {send "set global validate_password.policy=0;\r"}
expect "> " {send "set global validate_password.mixed_case_count=0;\r"}
expect "> " {send "set global validate_password.number_count=0;\r"}
expect "> " {send "set global validate_password.special_char_count=0;\r"}
expect "> " {send "set global validate_password.length=6;\r"}
expect "> " {send "alter user 'root'@'localhost' identified by '$new_passwd';\r"}
expect "#" {send "exit\r"}
EOF
echo "[Finish] 修改密码"
#修改防火墙配置
yum install -y iptables iptables-services
change_iptables
echo "[Finish] 修改防火墙配置"
#修改为UTF-8格式
sed -i 's/mysqld]/&\ncharacter_set_server=utf8/' /etc/my.cnf
sed -i "s/server=utf8/&\ninit_connect='SET NAMES utf8'/" /etc/my.cnf
systemctl restart mysqld
echo "[Finish] 修改为UTF-8格式"
#验证密码
expect << EOF
spawn mysql -uroot -p
expect ":" {send "$new_passwd\r"}
expect ">" {send "exit\r"}
EOF
echo "[Finish] 验证密码"
echo "Mysql setup finish and Password is '$new_passwd'"
#释放变量
unset old_passwd
unset new_passwd
```

## Shell:设置静态IP ##

```shell
#! /bin/bash
set -e #错误退出
clear
echo "**************************************************************"
echo "Shell:    $0               "
echo "版本:     1                "          
echo "日期:     2021-10-13       "
echo "描述:     用于静态IP的设置  "
echo "作者:     Wulve            "
echo "系统:     CentOS 7         "
echo "使用:     $0 [网卡名称]    "
echo "**************************************************************"
echo
echo "配置的网卡为: $1"
ifcfg_path=/etc/sysconfig/network-scripts/ifcfg-$1;
#恢复网络配置
if [ "$2" = "-r" ]; then
    echo "--------------------------------------------------------------"
    echo "准备恢复网络配置!"
    #判断网卡配置是否无误。
    if [ -f "$ifcfg_path.bak.ByWulve" ]; then
        cp $ifcfg_path.bak.ByWulve $ifcfg_path|| ! echo '恢复出错!' || exit
        echo "重启networkf服务"
        service network restart
        echo "重启networkf服务完成"
        echo "恢复完成!"
        echo "--------------------------------------------------------------"
        echo "当前网络配置信息:"
        ifconfig
        echo "--------------------------------------------------------------"
        exit
    else 
        echo "$1网卡配置备份文件不存在, 或者网卡有误。"
        ifconfig
        exit
    fi
fi
#判断网卡配置是否无误。
if [ -f "$ifcfg_path" ]; then
    echo "$1网卡配置存在"
else 
    echo "$1网卡配置不存在,请确认网卡名称。"
    ifconfig
    exit
fi
echo

#获取相关输入数据
echo "--------------------------------------------------------------"
echo "网卡配置文件: $ifcfg_path"
read -r -p "请输入静态IP    :" input_IPADDR
read -r -p "请输入子网掩码  :" input_NETMASK
if  [ ! -n "$input_NETMASK" ] ;then
    echo "子网掩码未配置,使用:255.255.255.0"
    input_NETMASK=255.255.255.0
fi
read -r -p "请输入网关地址  :" input_GATEWAY
if  [ ! -n "$input_DNS1" ] ;then
    echo "DNS未配置,使用:114.114.114.114"
    input_NETMASK=114.114.114.114
fi
read -r -p "请输入DNS地址   :" input_DNS1
if  [ ! -n "$input_DNS1" ] ;then
    echo "DNS未配置,使用:114.114.114.114"
    input_NETMASK=114.114.114.114
fi
#echo "--------------------------------------------------------------"
#测试使用
# input_IPADDR=192.168.129.128
# input_NETMASK=255.255.255.0
# input_GATEWAY=192.168.129.2
# input_DNS1=114.114.114.114
# echo "相关配置信息:"
# echo "静态IP    :$input_IPADDR" 
# echo "子网掩码  :$input_NETMASK" 
# echo "网关地址  :$input_GATEWAY" 
# echo "DNS地址   :$input_DNS1" 
#备份配置
echo "--------------------------------------------------------------"
echo "备份$1网卡配置"
if [ -f "$ifcfg_path.bak.ByWulve" ]; then
    echo "$1网卡备份配置已经存在, 本次不备份."
else 
    cp $ifcfg_path $ifcfg_path.bak.ByWulve
    echo "备份完成备份位置:$ifcfg_path.bak.ByWulve"
fi
read -r -p "配置错误将会断开SSH连接, 是否继续执行? [Y/N] " input
case $input in
    [yY][eE][sS]|[yY])
        echo "同意，继续执行"
        ;;
    [nN][oO]|[nN])
        echo "拒绝，执行退出"
        exit 1
        ;;
    *)
        echo "无效输入,执行退出"
        exit 1
        ;;
esac
#修改配置
echo "--------------------------------------------------------------"
echo "修改$1网卡配置"
sed -i "4c\BOOTPROTO="static" #修改为静态IP,默认为dhcp(已修改)" "$ifcfg_path"
sed -i "6c\IPV4_FAILURE_FATAL="yes" #开启IPV4(已修改)"         "$ifcfg_path" #开启IPV4
sed -i "7c\IPV6INIT="no" #关闭IPV6(已修改)"                    "$ifcfg_path" #关闭IPV6
sed -i "7c\#IPV6_ADDR_GEN_MODE="stable-privacy" #注释(已修改)" "$ifcfg_path" #注释第11行IPV6_ADDR_GEN_MODE="stable-privacy"
echo "IPADDR="$input_IPADDR" #设置的静态IP地址">>"$ifcfg_path"
echo "NETMASK="$input_NETMASK" #设置子网掩码">>"$ifcfg_path"
echo "GATEWAY="$input_GATEWAY" #设置网关地址">>"$ifcfg_path"
if  [ ! -n "$input_DNS1" ] ;then
    echo "未输入DNS地址，不配置"
else
    echo "DNS1="$input_DNS1" #设置DNS地址">>"$ifcfg_path"
fi
echo "$1网卡配置修改完成"
#重启network服务
echo "--------------------------------------------------------------"
echo "重启networkf服务"
service network restart
echo "重启networkf服务完成"
#ping www.baidu.com
echo "--------------------------------------------------------------"
echo "准备ping www.baidu.com 测试"
ping -c3 baidu.com
#输出相关配置
echo "--------------------------------------------------------------"
echo "相关配置信息:"
echo "静态IP    :$input_IPADDR" 
echo "子网掩码  :$input_NETMASK" 
echo "网关地址  :$input_GATEWAY" 
echo "DNS地址   :$input_DNS1" 
echo "静态配置完成, 如果使用有问题:"
echo "使用'$0 $1 -r'命令可以恢复初始网络配置"
echo "--------------------------------------------------------------"

```

## Shell:设置静态IP 2 ##

```shell

#! /bin/bash
set -e #错误退出
clear
echo "--------------------------------------------------------------
Shell:        $0                  
Version:      2                   
Data:         2021-10-13          
Description:  Settings static IP. 
Author:       Wulve               
System:       CentOS 7            
Instructions: $0 [Network card’s name]    
--------------------------------------------------------------
The network card being configured is: $1
正在配置的网卡是: $1"
ifcfg_path=/etc/sysconfig/network-scripts/ifcfg-$1;
#恢复网络配置
if [ "$2" = "-r" ]; then
    echo "--------------------------------------------------------------"
    echo "Prepare to restore network configuration"
    echo "准备恢复网络配置!"
    #判断网卡配置是否无误。
    if [ -f "$ifcfg_path.bak.ByWulve" ]; then
        cp $ifcfg_path.bak.ByWulve $ifcfg_path|| ! echo '恢复出错!' || exit
        echo "Restart networkf service"
        echo "重启networkf服务"
            service network restart
        echo "Restart the networkf service to complete！"
        echo "重启networkf服务完成"
        echo "Recovery is complete!"
        echo "恢复完成!"
        echo "--------------------------------------------------------------"
        echo "Current network configuration information:"
        echo "当前网络配置信息:"
            ifconfig
        echo "--------------------------------------------------------------"
        exit
    else 
        echo "$1 The network card configuration backup file does not exist, or the network card is incorrect."
        echo "$1网卡配置备份文件不存在, 或者网卡有误。"
        ifconfig
        exit
    fi
fi
#判断网卡配置是否无误。
if [ -f "$ifcfg_path" ]; then
    echo "$1 Network card configuration exists"
    echo "$1网卡配置存在"
else 
    echo "$1 The network card configuration does not exist, please confirm the network card name."
    echo "$1网卡配置不存在,请确认网卡名称。"
    ifconfig
    exit
fi
echo

#获取相关输入数据
echo "--------------------------------------------------------------"
echo "Network card configuration file location: $ifcfg_path"
echo "网卡配置文件位置: $ifcfg_path"
read -r -p "Please input the static IP (输入IP)    :" input_IPADDR             #设置静态IP
read -r -p "Please input the NETMASK (输入子网掩码):" input_NETMASK            #设置子网掩码
if  [ ! -n "$input_NETMASK" ] ;then
    echo "The NETMASK is null,that will be: 255.255.255.0"
    input_NETMASK=255.255.255.0
fi
read -r -p "Please input the GATEWAY (输入默认网关):" input_GATEWAY                #设置默认网关
read -r -p "Please input the DNS (输入DNS)         :" input_DNS1                       #设置DNS地址

#echo "--------------------------------------------------------------"
#测试使用
# input_IPADDR=192.168.129.128
# input_NETMASK=255.255.255.0
# input_GATEWAY=192.168.129.2
# input_DNS1=114.114.114.114
# echo "相关配置信息:"
# echo "静态IP    :$input_IPADDR" 
# echo "子网掩码  :$input_NETMASK" 
# echo "网关地址  :$input_GATEWAY" 
# echo "DNS地址   :$input_DNS1" 
#备份配置
echo "--------------------------------------------------------------"
echo "Backup $1 network card configuration
备份$1网卡配置"
if [ -f "$ifcfg_path.bak.ByWulve" ]; then
    echo "The backup configuration of the $1 network card already exists, and it will not be backed up this time."
    echo "$1网卡备份配置已经存在, 本次不备份."
else 
    cp $ifcfg_path $ifcfg_path.bak.ByWulve
    echo "The backup is complete, the backup location:$ifcfg_path.bak.ByWulve"
    echo "备份完成备份位置:$ifcfg_path.bak.ByWulve"
fi
echo "The configuration error will disconnect the SSH.Do you want to continue?[Y/N]"
read -r -p "配置错误将会断开SSH连接, 是否继续执行? [Y/N] " input
case $input in
    [yY][eE][sS]|[yY])
        echo "Agree, continue"
        echo "同意，继续执行"
        ;;
    [nN][oO]|[nN])
        echo "Reject, execute exit"
        echo "拒绝，执行退出"
        exit 1
        ;;
    *)
        echo "Invalid input, execute exit"
        echo "无效输入,执行退出"
        exit 1
        ;;
esac
#修改配置
echo "--------------------------------------------------------------"
echo "Modifying the configuration of the $1 network card
修改$1网卡配置"
sed -i 's/BOOTPROTO="dhcp"/BOOTPROTO="static"/g' "$ifcfg_path"
#sed -i "4c\BOOTPROTO="static" #修改为静态IP,默认为dhcp(已修改)" "$ifcfg_path"
sed -i 's/IPV4_FAILURE_FATAL="no"/IPV4_FAILURE_FATAL="yes"/g' "$ifcfg_path"
#sed -i "6c\IPV4_FAILURE_FATAL="yes" #开启IPV4(已修改)"         "$ifcfg_path" #开启IPV4
sed -i 's/IPV6INIT="yes"/IPV6INIT="no"/g' "$ifcfg_path"
#sed -i "7c\IPV6INIT="no" #关闭IPV6(已修改)"                    "$ifcfg_path" #关闭IPV6
sed -i 's/^IPV6_ADDR_GEN_MODE="stable-privacy"/#IPV6_ADDR_GEN_MODE="stable-privacy"/g' "$ifcfg_path"
#sed -i "7c\#IPV6_ADDR_GEN_MODE="stable-privacy" #注释(已修改)" "$ifcfg_path" #注释第11行IPV6_ADDR_GEN_MODE="stable-privacy"
echo " ">>"$ifcfg_path"
echo "IPADDR="$input_IPADDR" #设置的静态IP地址">>"$ifcfg_path"
echo "NETMASK="$input_NETMASK" #设置子网掩码">>"$ifcfg_path"
echo "GATEWAY="$input_GATEWAY" #设置网关地址">>"$ifcfg_path"
if  [ ! -n "$input_DNS1" ] ;then
    echo "DNS address is not entered, DNS is not configured"
    echo "未输入DNS地址，不配置DNS"
    input_DNS1="NULL"
else
    echo "DNS1="$input_DNS1" #设置DNS地址">>"$ifcfg_path"
fi
echo "$1 network card configuration has been modified."
echo "$1网卡配置修改完成"
#重启network服务
echo "--------------------------------------------------------------"
echo "Restart networkf service."
echo "重启networkf服务."
    service network restart
echo "Restart the network service to complete."
echo "重启networkf服务完成"
#ping www.baidu.com
echo "--------------------------------------------------------------"
echo "Prepare to ping www.baidu.com to test."
echo "准备ping www.baidu.com 测试"
ping -c3 baidu.com
#输出相关配置
echo "--------------------------------------------------------------
Related configuration information:
相关配置信息:
Static IP(静态IP) :$input_IPADDR
NETMASK(子网掩码) :$input_NETMASK
GATEWAY(网关地址) :$input_GATEWAY
DNS(DNS地址)      :$input_DNS1
Use the'$0 $1 -r' command to restore the initial network configuration
使用'$0 $1 -r'命令可以恢复初始网络配置
--------------------------------------------------------------"

```

## Shell:JDK配置 ##

```shell
#!/bin/bash
#set -e #错误退出
clear
echo "**************************************************************"
echo "Shell:    $0               "
echo "版本:     1                "          
echo "日期:     2021-10-13       "
echo "描述:     用于JDK的配置  "
echo "作者:     Wulve            "
echo "系统:     CentOS 7         "
echo "使用:     $0 [包路径]    "
echo "**************************************************************"
#获取输入路径,如果是相对路径转为绝对路径
input_path="$1"
if [ ${input_path:0:1} = "/"  ]; then
    pakge_path=$input_path
else
    pakge_path="$(pwd)/${input_path##*/}"
fi
#判断文件是否存在
if [ -f "$pakge_path" ]; then
    echo "$pakge_path 文件存在"
else 
    echo "$pakge_path 文件不存在"
    exit
fi
java -version
if [ $? -ne 0 ];then #如果上一个命令退出没有数据，则执行
    mkdir /usr/local/java
    cp $pakge_path /usr/local/java
    cd /usr/local/java && tar -xvf $pakge_path
    echo "export JAVA_HOME=/usr/local/java/jdk1.8.0_251" >> /etc/profile
    echo 'export JRE_HOME=$JAVA_HOME/jre' >> /etc/profile
    echo 'export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin' >> /etc/profile
    echo 'export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib' >> /etc/profile
    
    source /etc/profile
else
    echo "JDK is instelled."
    exit
fi
```

## Shell:CPU占用率高的进程 ##

```shell
#!/bin/bash
#topk 前k个进程
#samplingTime 采样时间, 单位秒
#interval 采样间隔时间, 单位秒

TOPK={{topk}}
SECS={{samplingTime}}
INTERVAL={{interval}}

STEPS=$(( $SECS / $INTERVAL ))
TEMP_FILE_PREFIX="/tmp/tat_public_cpu_usage"
echo Watching CPU usage...
for((i=0;i<$STEPS;i++))
do
  ps -eocomm,pcpu | tail -n +2 >> $TEMP_FILE_PREFIX.$$
  sleep $INTERVAL
done
echo
echo CPU eaters :
cat $TEMP_FILE_PREFIX.$$ | \
awk '
{ process[$1]+=$2;}
END{
  for(i in process) {
    printf("%-20s %s\n",i, process[i]) ;
  }
}' | sort -nrk 2 | head -n $TOPK
rm $TEMP_FILE_PREFIX.$$
```

## Shell:显示僵尸进程 ##

```shell
#!/bin/bash
processes=$(ps ax -o user,pid,ppid,pgid,args,stat,start,time)
zombies=$(echo -e "${processes}" | grep -E "\s(Z|z|Z.*)\s")
if [ $? == 1 ]; then
  echo "no zombie processes exists on machine"
else
  echo -e "${processes}" | head -1
  echo "$zombies"
fi
```

## Shell:清理磁盘 ##

```shell
#!/bin/bash
#批量在多台Linux实例上清理磁盘
#delayTime  7d  文件的有效时间。如 
#          7d（代表7天），1h（代表1小时），5m（代表5分钟），默认是7d
#filePath          清理文件路径。如：/root/log/
#matchPattern   清理文件匹配格式，如 *.log。 支持正则匹配

  
function deletefiles() {
  if [ ! -d $2 ]; then
    echo "The specified directory("$2") is not exist."
    return
  fi
  
  expiredTimeUnit=${1: -1}
  expiredTimeValue=${1:0:-1}
  
  if [ "$expiredTimeUnit" = "d" ]; then
    expiredTime=$(($expiredTimeValue * 24 * 60 * 60))
  elif [ "$expiredTimeUnit" = "h" ]; then
    expiredTime=$(($expiredTimeValue * 60 * 60))
  elif [ "$expiredTimeUnit" = "m" ]; then
    expiredTime=$(($expiredTimeValue * 60))
  else
    echo "The unit("$expiredTimeUnit") of file age is not supported."
    return
  fi
  
  for file in $(find $2 -type f -name "$3"); do
    local currentDate=$(date +%s)
    local modifyDate=$(stat -c %Y $file)
    local existTime=$(($currentDate - $modifyDate))
  
    if [ $existTime -gt $expiredTime ]; then
      echo "File cleaning succeeded,path:"$file"."
      rm -f $file
    fi
  done
}
  
deletefiles {{delayTime}} {{filePath}} "{{matchPattern}}"
```

## Shell:查看占用空间 ##

```shell
#!/bin/bash
#查看目录占用磁盘空间大小
#directory 目标目录

du -sh {{directory}}
```

## Shell:批量安装或卸载包 ##

```shell
#!/bin/bash
#批量在多台Linux实例上安装或卸载包。
#Installer  apt-get  
#     包管理器,只允许输入以下值[apt-get,yum]
#Action  install        
#     安装或卸载 package,只允许输入以下值[install,uninstall]
#packageName
#     yum/apt-get   安装的包名

function configurePackages() {
    installer=$1
    action=$2
    packageName=$3
    if [ "$installer" = "yum" ]; then
        if [ "$action" = "install" ]; then
            yum install -y $packageName
            if [ $? -ne 0 ]; then
                echo "Package install failed. Please check your command"
                exit 1
            fi
        elif [ "$action" = "uninstall" ]; then
            yum remove -y $packageName
            if [ $? -ne 0 ]; then
                echo "Package uninstall failed. Please check your command"
                exit 1
            fi
        else
            echo "Package command must be install or uninstall"
            exit 1
        fi
    elif [ "$installer" = "apt-get" ]; then
        if [ "$action" = "install" ]; then
            apt-get -y install $packageName
            if [ $? -ne 0 ]; then
                echo "Package install failed. Please check your command"
                exit 1
            fi
        elif [ "$action" = "uninstall" ]; then
            apt-get -y remove $packageName
            if [ $? -ne 0 ]; then
                echo "Package uninstall failed. Please check your command"
                exit 1
            fi
        else
            echo "Package command must be install or uninstall"
            exit 1
        fi
    else
        echo "Unknown package installer. Only support yum/apt-get"
        exit 1
    fi
}
  
configurePackages {{installer}} {{action}} {{packageName}}
```

>>>>>>> 74a2d3c9d9df3854d1f22d1c7136d23a2d5ba9b3
