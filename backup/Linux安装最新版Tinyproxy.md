#前言
之前在Linux服务器安装过tinyproxy用于ip代理，但是yum安装的话，版本最高只到1.8.3，是不支持账号密码验证的，也就是说要么限制ip，不然任何人都能连。而到了1.10.0就支持Basic HTTP Authentication了，但是得通过编译安装。目前Github上最新的是1.11.1，本文是安装的简要教程。
#安装及配置
如果通过yum安装过1.8.3版本，得先卸载掉。
`yum erase tinyproxy`
以下是从Github下载安装最新版的命令。
`# 获取安装包
wget https://github.com/tinyproxy/tinyproxy/releases/download/1.11.0/tinyproxy-1.11.0.tar.gz
 
# 解压安装包
tar -zxvf tinyproxy-1.11.0.tar.gz
 
# 进入安装文件夹
cd tinyproxy-1.11.0
 
# 编译并安装
./configure
make
make install`
安装完成后可用以下命名查看路径和版本。