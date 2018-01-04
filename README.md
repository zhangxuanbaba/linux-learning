# linux的基础配置
(小编的linux系统是CentOS7.2的) <br>  
[1.修改主机名称](#修改主机名称) <br>  
[2.配置iptables](#配置iptables) <br>   
[ 3.安装jdk](#安装jdk)  <br>  
[4.安装tomcat](#安装tomcat) 


## 修改主机名称

临时生效修改：使用命令行修改 hostname 主机名(可自定义)，重新登录 shell 生效，服务器重启后失效。<br>

#### centOS 7以上的linux系统（包含7）
   使用命令hostnamectl set-hostname 主机名 来修改，修改完毕后重新 SHELL 登录即可。

#### centOS 7以下的linux系统
   
   1.以根用户登录，或者登录后切换到根用户，然后在提示符下输入hostname命令，可以看出当前系统的主机名为localhost.localdomain。
   
   2.更改/etc/sysconfig下的network文件，在提示符下输入vim /etc/sysconfig/network，然后将HOSTNAME后面的值改为想要设置的主机名。
   
   3.更改/etc下的hosts文件，在提示符下输入vim /etc/hosts，然后将localhost.localdomain改为想要设置的主机名。
   
   4.在提示符下输入reboot命令，重新启动服务器。
   
   5.重启完成后用hostname命令查询系统主机名，可以看出系统主机名已经变更为你设定的名字。
 
 ## 配置iptables
   在centOS 7 以上的系统中默认使用的是firewall,centOS 7以下的系统使用的是iptables <br>
   firewall和iptables的区别：firewall是centos7里面的新的防火墙命令，它底层还是使用 iptables 对内核命令动态通信包过滤的，简单理解就是firewall是centos7下管理iptables的新命令。

