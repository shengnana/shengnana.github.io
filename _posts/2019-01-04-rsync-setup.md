## RSYNC安装 ##
#### 检查rsync是否已经安装 ####
  {% raw %}
  rpm -qa | grep rsync  //或者使用whereis rsync也可以
  yum install rsync     // 如果未安装则安装，mac可以使用brew
  {% endraw %}
  
#### 配置config文件 ####

  {% raw %}
  mkdir /etc/rsyncd
  touch /etc/rsyncd/rsyncd.conf         // 主配置文件
  touch /etc/rsyncd/rsyncd.secrets      // 用户名密码文件，一组用户一行，用户名和密码使用 : 分割
  chmod 0600 /etc/rsyncd/rsyncd.secrets  // rsyncd服务的密码文件权限必须是600
  touch /etc/rsyncd/rsyncd.motd          // 连接上rsync服务器显示的欢迎信息，可以不填
  {% endraw %}
  

