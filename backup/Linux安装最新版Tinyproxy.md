# 前言
之前在Linux服务器安装过tinyproxy用于ip代理，但是yum安装的话，版本最高只到1.8.3，是不支持账号密码验证的，也就是说要么限制ip，不然任何人都能连。而到了1.10.0就支持Basic HTTP Authentication了，但是得通过编译安装。目前Github上最新的是1.11.1，本文是安装的简要教程。
# 安装及配置
如果通过yum安装过1.8.3版本，得先卸载掉。
`yum erase tinyproxy`
以下是从Github下载安装最新版的命令。
`获取安装包
`wget https://github.com/tinyproxy/tinyproxy/releases/download/1.11.0/tinyproxy-1.11.0.tar.gz`
 
 解压安装包
`tar -zxvf tinyproxy-1.11.0.tar.gz`
 
进入安装文件夹
`cd tinyproxy-1.11.0`
 
编译并安装
./configure
make
make install`
安装完成后可用以下命名查看路径和版本。
`# 查看路径
which tinyproxy
/usr/local/bin/tinyproxy
 
查看版本，最新1.11.0
tinyproxy -v 
tinyproxy 1.11.0`

修改配置文件，下面是安装后配置文件位置。

`vi /usr/local/etc/tinyproxy/tinyproxy.conf`
基本的配置可以见之前那篇文章，主要说下不同。

```
# 将下面直接注释掉，允许所有ip访问
#Allow 127.0.0.1
#Allow ::1
 
# 设置用户名密码
BasicAuth username password
 
# 顺便将下面两行取消注释，后面有用到
PidFile "/usr/local/var/run/tinyproxy/tinyproxy.pid"
LogFile "/usr/local/var/log/tinyproxy/tinyproxy.log"
```
保存配置文件后，使用下面的命令启动。

```
# 指定配置文件启动（后台启动）
tinyproxy -c /usr/local/etc/tinyproxy/tinyproxy.conf
```
启动后就可以测试下

```
# 不加验证参数不会正常返回
curl -x http://127.0.0.1:8888 www.baidu.com
 
# 正常返回
curl -x http://username:password@127.0.0.1:8888 www.baidu.com
```
以上就是简要安装与配置，但是为了方便管理服务，得添加到系统服务。

##添加到系统服务
```
# 新建tinyproxy.service文件
vi /usr/lib/systemd/system/tinyproxy.service
 
# 贴入以下代码
[Unit]
Description=Startup script for the tinyproxy server
After=network.target
 
[Service]
Type=forking
PIDFile=/usr/local/var/run/tinyproxy/tinyproxy.pid
ExecStart=/usr/local/bin/tinyproxy -c /usr/local/etc/tinyproxy/tinyproxy.conf
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
 
[Install]
WantedBy=multi-user.target
```
上面的配置文件直接仿照yum安装的1.8.3版的，会出现点问题，启动不了，因为tinyproxy配置文件里的log和pid文件不存在，得先通过以下命令创建。

```
mkdir -p /usr/local/var/run/tinyproxy
mkdir -p /usr/local/var/log/tinyproxy
touch /usr/local/var/log/tinyproxy/tinyproxy.log
touch /usr/local/var/run/tinyproxy/tinyproxy.pid
```
然后就可以正常使用service命令了。

```
service tinyproxy start    # 启动
service tinyproxy stop     # 停止
service tinyproxy restart  # 重启
service tinyproxy status   # 状态
 
# 将tinyproxy服务设置开机自启
sudo systemctl enable tinyproxy
```
最后整理下命令：

```
# 复制以下命令，直接到修改tinyproxy.conf
yum erase -y tinyproxy
wget https://github.com/tinyproxy/tinyproxy/releases/download/1.11.0/tinyproxy-1.11.0.tar.gz
tar -zxvf tinyproxy-1.11.0.tar.gz
cd tinyproxy-1.11.0
./configure
make
make install
vi /usr/local/etc/tinyproxy/tinyproxy.conf
 
# 修改后，使用以下命令，到编辑tinyproxy.service
mkdir -p /usr/local/var/run/tinyproxy
mkdir -p /usr/local/var/log/tinyproxy
touch /usr/local/var/log/tinyproxy/tinyproxy.log
touch /usr/local/var/run/tinyproxy/tinyproxy.pid
vi /usr/lib/systemd/system/tinyproxy.service
 
# 粘贴以下代码
====
 
[Unit]
Description=Startup script for the tinyproxy server
After=network.target
 
[Service]
Type=forking
PIDFile=/usr/local/var/run/tinyproxy/tinyproxy.pid
ExecStart=/usr/local/bin/tinyproxy -c /usr/local/etc/tinyproxy/tinyproxy.conf
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
 
[Install]
WantedBy=multi-user.target
 
====
 
# 保存后，再使用以下命令启动tinyproxy
sudo systemctl enable tinyproxy
service tinyproxy start
service tinyproxy status

```