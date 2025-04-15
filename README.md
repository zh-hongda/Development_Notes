# 开发笔记

---
- [开发笔记](#开发笔记)
- [Conda常用命令](#conda常用命令)
- [Linux常用命令](#linux常用命令)
  - [防火墙](#防火墙)
  - [用户管理](#用户管理)
  - [打包、压缩和解压缩文件](#打包压缩和解压缩文件)
  - [时区](#时区)
  - [查看硬件环境](#查看硬件环境)
  - [查看cpu资源占用情况](#查看cpu资源占用情况)
  - [打印用户执行过的命令](#打印用户执行过的命令)
  - [后台运行](#后台运行)
  - [设置文件的权限](#设置文件的权限)
  - [Linux下统计当前文件夹下的文件个数、目录个数](#linux下统计当前文件夹下的文件个数目录个数)
  - [截取文件前几行](#截取文件前几行)
  - [指定可见显卡](#指定可见显卡)
  - [Ubuntu修改默认python版本](#ubuntu修改默认python版本)
  - [SSH](#ssh)
    - [安装SSH](#安装ssh)
  - [ssh常用命令](#ssh常用命令)
- [Docker](#docker)
  - [常用命令](#常用命令)
  - [Docker \& OpenCV](#docker--opencv)
  - [Docker远程桌面](#docker远程桌面)
- [VsCode](#vscode)
  - [Vscode \& SSH](#vscode--ssh)
  - [在VsCode中推送代码到github](#在vscode中推送代码到github)
- [Nvidia](#nvidia)
  - [安装显卡驱动](#安装显卡驱动)
  - [显卡常用命令](#显卡常用命令)
- [Latex](#latex)
  - [Table](#table)
  - [Figure](#figure)
  - [Algorithm](#algorithm)
- [Matlab](#matlab)
  - [ubuntu安装matlab](#ubuntu安装matlab)
  - [服务器跑matlab代码注意事项](#服务器跑matlab代码注意事项)
- [数据集相关代码](#数据集相关代码)
  - [图片信息统计](#图片信息统计)
  - [数据集图片处理](#数据集图片处理)
  - [bash脚本对应的python代码](#bash脚本对应的python代码)
- [Final](#final)

---


# Conda常用命令

```bash
# 创建一个python环境
 conda create --name 环境名 python=3.6

 # 列出所有虚拟环境
 conda info --env

 # 删除一个python环境
 conda env remove -n 环境名
 
  # conda 修改虚拟环境名称
 conda create --name newName（新环境名） --clone oldName（旧环境名）
 conda remove --name oldName（旧环境名） --all
 
# set the auto_activate_base parameter to false:
 conda config --set auto_activate_base false

 # 清理缓存, 删除没有用的包
 conda clean -p

 # 安装指定版本的package
 conda install numpy=1.16.2

 # 如果已经在激活环境下了, 那就可以直接使用conda list列出来的包就可以查看环境下已经安装包列表
 conda list

 # conda批量导出包含环境中所有组件的requirements.txt文件
 conda list -e > requirements.txt

 # 在虚拟环境中使用以下命令将当前虚拟环境中的依赖包以及版本号生成至文件中
 pip freeze > requirements.txt

 # 当要创建这个虚拟环境的完全副本, 可以创建一个新的虚拟环境, 并在其上运行以下命令：
 pip install -r requirements.txt

 -------------------------------------------------
 #修改conda镜像源
 # 查看现有的镜像源
 conda config --show channels

 # 添加镜像源
 conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/

 # 删除不想用的镜像源
 conda config --remove channels https://mirrors.tuna.tsinghua.edu.cn/tensorflow/linux/cpu/

 # 设置搜索时显示通道地址
 conda config --set show_channel_urls yes

 # 确定镜像是否添加成功, 
 conda config --show channels

 -------------------------------------------------
 # 一键修改国内镜像源, https://github.com/matpool/matools
 git clone https://github.com/matpool/matools.git 
 bash ~/matools/mirrors/switch_apt_source.sh 
 bash ~/matools/mirrors/switch_conda_source.sh
 bash ~/matools/mirrors/switch_pip_source.sh

 -------------------------------------------------
 # 更新至最新版本, 也会更新其它相关包
 conda update conda

 # 更新所有包
 conda update --all
 -------------------------------------------------
 #jupyter
 #https://blog.inkuang.com/2019/319/
 #首先切换到想要在 Jupyter Notebook 里使用的虚拟环境：
 conda activate 环境名称

 #安装 ipykernel：
 conda install ipykernel

 #写入 Jupyter 的 kernel,在当前虚拟环境里执行：(“环境名称”为当前虚拟环境的名称, 最后面引号内的字符串是该虚拟环境显示在 Jupyter Notebook 界面的名字, 可以随意修改。)
 python -m ipykernel install --user --name 环境名称 --display-name "Python (环境名称)"

 #切换 kernel,上面两步后就完成了 Jupyter 的相关配置, 启动 Jupyter Notebook：
 jupyter notebook

 #删除 kernel 环境,上面写入 kernel 的配置并不会随虚拟环境的删除而删除。也就是说即使删除了该虚拟环境, Jupyter Notebook 的界面上仍会有它的选项, 只是无法正常使用。此时就需要去手动删除 kernel 环境了：
 jupyter kernelspec remove 环境名称
```

# Linux常用命令

## 科学上网

```bash
# 下载clash-verge-rev
https://github.com/clash-verge-rev/clash-verge-rev
# 安装
sudo apt-get install ./clash-verge_1.7.7_amd64.deb
# 终端代理，仅在当前终端生效
export http_proxy="http://your-proxy:port"
export https_proxy="http://your-proxy:port"
```



## 防火墙

Ubuntu一般都默认安装了UFW（Uncomplicated Firewall）, 它是一款轻量化的工具, 主要用于对输入输出的流量进行监控。如果没有安装, 请用下面的命令安装：

```bash
# 安装ufw
apt install ufw

#查看版本信息
ufw version  

# 查看防火墙状态
ufw status
ufw status numbered

# 删除某个端口或者某条规则
ufw delete 端口号/数字

# 设置防火墙规则为“默认拒绝外来访问”
ufw default deny

# 修改防火墙规则：允许外部ip访问本机的22（ssh）端口。ufw打开或者关闭某个端口的命令为：ufw allow/deny 端口号。
ufw allow 22

# 启用防火墙
ufw enable

# 禁用防火墙
ufw disable

#查看防火墙策略
ufw status verbose 


https://blog.csdn.net/qq_54947566/article/details/124426713https://blog.csdn.net/cljdsc/article/details/120832554
```



## 用户管理

```bash
# 新建用户
sudo adduser hongda

# 赋予管理员权限
sudo usermod -aG sudo hongda

sudo userdel 用户名
deluser --remove-home newuser

for u in `cat /etc/shadow | cut -d":" -f1`;do crontab -l -u $u;done
```



## 打包、压缩和解压缩文件

```bash
# 压缩
tar -zcvf 压缩后的名字.tar.gz 待压缩的文件夹

# 解压缩
tar -zxvf 待解压的文件.tar.gz

# 参数的含义
-z, 调用gzip执行压缩或解压缩。
-c, 创建新的tar文件
-x, 解开tar文件
-v, 列出每一步处理涉及的文件的信息
-f, 指定要处理的文件名。
```



## 时区

```bash
apt-get update
# https://blog.csdn.net/liumiaocn/article/details/89184511
sudo apt-get install tzdata
```



## 查看硬件环境

```bash
# 总核数 = 物理CPU个数 X 每颗物理CPU的核数 
# 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l

#查看CPU信息（型号）
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c

#查看内存信息
cat /proc/meminfo
free -m

#查看所有存储设备
lsblk # list block device

#列出档案系统的整体磁盘使用量
df -h
lsblk

# 查看内存槽的数目、哪个槽位插了内存以及内存的大小。
sudo dmidecode|grep -P -A5 "Memory\s+Device"|grep Size|grep -v Range

# 查看当前目录和文件所占容量,Disk Usage
du -h -d 0 ./* # *表示当前目录所有文件
```



## 查看cpu资源占用情况

```bash
#设置信息更新周期为3
top -d 3
#查看多核CPU命令,在已经使用top命令打印出cpu信息后,按数字“1”可监控每个逻辑CPU的状况
#杀死指定用户所有进程
killall -u usrname
```



## 打印用户执行过的命令

```bash
#!/bin/bash
for i in /home/* ;
do  printf "next user "$i"\n" >> history.txt; sudo cat $i/.bash_history >> history.txt;
done
```



## 后台运行

```bash
# nohup命令可以在控制台关掉(退出帐户时)之后继续运行相应的进程。nohup就是不挂起的意思( no hang up)。
nohup command > myout.file 2>&1 &
# nohup命令配合 & 命令,在命令后面加上& 实现后台运行。例如：sh test.sh &
# 如果放在后台运行的作业会产生大量的输出到屏幕,最好使用下面的方法把它的输出重定向到某个文件中：
command>out.file 2>&1  & 
# 2>&1 是将标准出错重定向到标准输出, 这里的标准输出已经重定向到了out.file文件
```



## 设置文件的权限

```bash
$ sudo chmod 755 /etc/init.d/svnd.sh (注意一定要设置权限, 不然开机不会启动)
# u [代表所属用户] g [代表所属用户组] o [代表其他用户]
# 第二种设置文件权限的方法, 对于文件夹可以加"-R"选项, 即可递归处理所有文件

chmod u=rwx 文件名
chmod g=r 文件名
```



## Linux下统计当前文件夹下的文件个数、目录个数

```bash
#查看当前目录下的文件数量（不包含子目录中的文件）
ls -l|grep "^-"| wc -l

#查看当前目录下的文件数量（包含子目录中的文件） 注意：R, 代表子目录
ls -lR|grep "^-"| wc -l

#查看当前目录下的文件夹目录个数（不包含子目录中的目录）, 同上述理, 如果需要查看子目录的, 加上R
ls -l|grep "^d"| wc -l

#查询当前路径下的指定前缀名的目录下的所有文件数量,例如：统计所有以“20161124”开头的目录下的全部文件数量
ls -lR 20161124*/|grep "^-"| wc -l

#对每个命令参数做一下说明备注： 
ls -l #该命令表示以长列表输出指定目录下的信息（未指定则表示当前目录）, R代表子目录中的“文件”, 这个“文件”指的是目录、链接、设备文件等的总称
grep "^d"表示目录, "^-"表示文件
wc -l #表示统计输出信息的行数, 因为经过前面的过滤已经只剩下普通文件, 一个目录或文件对应一行, 所以统计的信息的行数也就是目录或文件的个数
```



## 截取文件前几行

```bash
head -100 a.txt > b.txt
```



## 指定可见显卡

```bash
export CUDA_VISIBLE_DEVICES=6,7
```



## Ubuntu修改默认python版本

查看系统中有哪些Python版本, 当前默认python版本

```bash
ls /usr/bin/python*
python --version
```

基于update-alternatives, 修改默认python版本

```bash
# 首先列出所有可用的python替代版本信息：
update-alternatives --list python

# 然后将Python的替代版本添加进去：
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.5 2

# 再次列出可用的Python替代版本：
update-alternatives --list python

# 在列出的Python替代版本中任意切换：
update-alternatives --config python

# 查看修改后的默认python版本
python --version

# 当系统不再存在某个Python替代版本时, 我们可以将其从update-alternatives列表中删除掉：
sudo update-alternatives --remove python /usr/bin/python2.7 
```

常见错误：

```bash
update-alternatives: 错误: 无 python 的候选项
# 如果出现以上所示的错误信息, 表示update-alternatives没有添加Python的替代版本
```

命令解释：

```bash
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
```

`update-alternatives` 主要用于管理系统中多个同类程序的默认版本。本质上就是管理软链接, 但提供了更规范安全的操作接口。

➢ `usr/bin/python` 是要创建的软链接的名字, 是几个版本共用的；

➢ 后面的 python 即服务名, 添加的版本会加入到名叫`python`的这个版本系列里, 如果之前不存在（“无候选项”）则创建；

➢ 接下来的 `/usr/bin/python2` 就是软件的实际位置；

➢ 最后的数字是优先级, 后续可以选择自动模式和手动模式, 自动模式下就会自动选择优先级值最大的一个版本。

## SSH

### 安装SSH

```bash
# 为root账号设置密码
passwd root

# 替换国内源
sed -i s:/archive.ubuntu.com:/mirrors.aliyun.com:g /etc/apt/sources.list
sed -i s:/security.ubuntu.com:/mirrors.aliyun.com:g /etc/apt/sources.list

# 很多镜像都不会默认安装 ssh, 所以需要在容器内安装 ssh 服务：
apt update && apt install -y --no-install-recommends openssh-server

# 一般进入容器时使用的都是 root 账号, 但是 ssh 默认是禁止 root 账号使用密码远程登录的, 所以需要修改 ssh 配置文件使其允许：(但是如果你启动容器的时候使用 -u 参数指定了一个非 root 用户, 那么这步可以跳过。)
echo 'PermitRootLogin yes'>>/etc/ssh/sshd_config

# 启动 ssh 服务：
service ssh start
```



## ssh常用命令

```bash
sudo service ssh start

sudo service ssh stop

sudo service ssh restart

sudo service ssh status

# 从本机复制文件到远程, scp -r 本地文件路径 远程用户名@远程ip:目标地址路径
scp -r /data/hongda/data/environmentPackage/start_image.sh  hongda@192.168.1.175:/data/hongda/environment
```



# Docker

## 常用命令

```bash
# docker 安装完毕后, 普通用户是无法使用docker的, 所以需要将普通用户添加到docker组
# 添加 docker 用户组
groupadd docker
# 把需要使用docker的用户添加进该组, 这里是hongda
gpasswd -a hongda docker

# 更新用户组
newgrp docker
------------------------------------------------

# 重启 docker
systemctl restart docker

# 查看docker的运行状态
systemctl status docker

# 查看镜像列表
docker images

# 查看容器列表
docker ps

# 使用镜像创建一个容器
docker run --privileged -d -it -p 11199:22 -p 11900:5900 --gpus all --name=test --mount type=bind,source=/data/hongda,target=/home/hongda gotoeasy/ubuntu-desktop

# docker stop 容器ID或容器名
# 参数 -t：关闭容器的限时, 如果超时未能关闭则用kill强制关闭, 默认值10s, 这个时间用于容器的自己保存状态
docker stop -t=60 容器ID或容器名

# 在主机和容器间传输文件
docker cp woalsdnd-isbi_2018_fundus_challenge-c51a234a6870 vrt:/root/
docker cp 主机文件的路径 容器名字：拷贝到的目的地路径

# docker 将当前容器保存为镜像
docker commit -a "runoob.com" -m "my apache" 容器名称或id 打包的镜像名称:标签
# OPTIONS说明：
# -a :提交的镜像作者；
# -c :使用Dockerfile指令来创建镜像；
# -m :提交时的说明文字；
# -p :在commit时, 将容器暂停。
# example:
docker commit -a "hongda" -m "matlab2018a + pytorch1.8.1" 70b947689e76 ubuntu:v1.0

# 将镜像保存为tar文件, docker save -o 要保存的文件名  要保存的镜像
docker save -o ubuntu.tar ubuntu:v1.0

# docker 从 tar 文件加载镜像
docker load --input ubuntu.tar

# docker重命名镜像名称和tag
docker tag 70ff7873d7cd my_centos:tomcat-centos

# 查看docker占用的存储空间
docker system df
```



## Docker配置代理

```bash
# docker配置代理
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/http-proxy.conf <<EOF
[Service]
Environment="HTTP_PROXY=http://your-proxy:port"
Environment="HTTPS_PROXY=http://your-proxy:port"
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## Docker & OpenCV

```bash
# libGL.so.1: cannot open shared object file: No such file or directory
pip install opencv-python-headless
```



## Docker远程桌面

```bash
参考链接：https://blog.csdn.net/weixin_42068573/article/details/131227544

apt update && apt install -y sudo

# 新建用户, 并切换为新用户
adduser developer

usermod -aG sudo developer

su developer

sudo passwd root

# 安装VNC和xfce4
sudo apt install xfce4 tigervnc-standalone-server

# xcfe启动需要
sudo service dbus start

# 设置VNC密码
vncpasswd

# 创建并编辑VNC配置文件
vim ~/.vnc/xstartup

# 按i键进入编辑模式, 并将配置文件修改为如下内容。
#!/bin/sh
xsetroot -solid grey
startxfce4 &

# 启动VNCserver, vncserver -geometry 1920x1080 :1
vncserver -localhost no :1

# 运行以下命令, 关闭已启动的VNC。
vncserver -kill :1

# 打开终端报错：failed to execute default Terminal Emulator
# 解决办法：
sudo apt-get install xfce4-terminal && sudo apt purge gnome-terminal

# 然后Applications - Settings - Perfered Applications - Utilities - Terminal Emulator选择Xfce Terminal即可, 重新打开终端正常执行。

# 安装火狐浏览器,依照终端一样的操作, 选择firefox浏览器为默认浏览器, chrome、edge不稳定, 容易宕机。：
sudo apt install firefox
```





# VsCode

## Vscode & SSH

```
Host example_1
    HostName 180.167.195.118
    port 4722
    User hongda

Host example_2
    HostName 192.168.10.47
    port 11163
    User hongda
    ProxyCommand ssh -W %h:%p fudan_4

Host jetson-xavier-nx
    HostName 192.168.3.67
    port 22
    User developer

Host jetson-xavier-nx-fire
    HostName 192.168.3.67
    port 11190
    User root
    ProxyCommand ssh -W %h:%p jetson-xavier-nx
```





## 在VsCode中推送代码到github

1. 建立远程仓库, 并将当前设备的SSH keys 添加到github中, 生成密钥`ssh-keygen -t rsa -C "hongdaaaaaaaa@163.com`，github网页版中添加密钥

2. 配置本地git环境

➢ 安装git, 并执行：

```bash
git config --global user.name "hongda" git config --global user.email "hongdaaaaaaaa@163.com"
```

➢ 安装git, 并执行：

```bash
git config --global user.name "hongda" git config --global user.email "hongdaaaaaaaa@163.com"
```

3. 拉取远程仓库到本地

```bash
git pull 
```

- 打开VSCode, 并在其中打开刚才拉取的项目文件夹

- 编辑（修改/保存）文件
- 在VScode中点击按钮"stage all changes", “commit all”

```bash
git commit -a -m "message"
```



4. 将本地仓库的内容推送到远程仓库，在VSCode的终端中输入:

```
# 添加远程仓库地
git remote add origin git@github.com: zh-hongda/仓库名.git 
# 将本地仓库推送至远程仓库, git push 远程仓库名 本地仓库分支名:远程仓库分支名
git push origin master:远程仓库分支名
```

5. 忽略指定文件, 需要在提交前写好.gitignore



## VsCode创建markdown目录

```bash
# 安装插件markdown all in one
# 基于目前的标题建立目录，如果新建了一个新的标题，可以保存，保存之后目录就更新了
ctrl+shift+p调出命令框输入命令 markdow
```





# Nvidia

## 安装显卡驱动

```bash
# 先查看是否已经有显卡驱动
nvidia-smi
# https://blog.csdn.net/qq_21768483/article/details/108824081
# 如果有, 卸载原先驱动
sudo apt-get remove --purge nvidia*

# 如果没有, 
apt-get update
apt install ubuntu-drivers-common
#查看当前显卡合适的驱动
ubuntu-drivers devices

#安裝驱动, 注意改成你自己合适的驱动
apt-get install nvidia-driver-450-server

# 查看驱动是否安装成功
nvidia-smi
```

## 显卡常用命令

```bash
# 清空显存占用
fuser -vk /dev/nvidia*

# 临时设置可见显卡
export CUDA_VISIBLE_DEVICES=0

# 设置GPU的Persistence Mode：
sudo nvidia-smi -pm 1

# 批量清理显卡中残留进程：
sudo fuser -v /dev/nvidia* |awk '{for(i=1;i<=NF;i++)print "kill -9 " $i;}' | sudo sh
# 清理指定GPU显卡中残留进程, 如GPU 2：
sudo fuser -v /dev/nvidia2 |awk '{for(i=1;i<=NF;i++)print "kill -9 " $i;}' | sudo sh
```

## Jetson环境配置

```bash
# 环境配置
https://docs.nvidia.com/sdk-manager/install-with-sdkm-jetson/index.html
https://blog.csdn.net/qq_44998513/article/details/133754589
```



# Latex

## Table

```latex
%三线表
\usepackage{booktabs} %制作表格时使用“booktabs”包。
\usepackage{color} %调整文字颜色时使用“color”包
%
\begin{table}[htbp] %指定表格位置, “h” here, “t” top, “b” bottom, “p”
    \vspace{-1cm} %调整与上文的垂直距离, 不仅适用于表格。
    \centering    %使表格居中
    \caption{Accuracy and speed comparisons on the test dataset. The best two results are shown in red and blue fonts, respectively.} %指定标题
    \begin{tabular}{cccccc} %指定一共多少列, 一个“c”表示一列。也可用“l”或“r”替换, 分布控制表格内容左对齐和右对齐。
        \toprule  %第一条线的规则
        & KCF & SAMF & C-COT & ECO-HC & PSCT\\
        \midrule  %第二条线的规则
        Success & 0.458 & 0.568 & 0.695 & \textcolor{blue}{0.704} & \textcolor{red}{0.792}\\ %控制表格中文字颜色
        Precision & 0.816 & 0.834 & 0.870 & \textcolor{blue}{0.900} & \textcolor{red}{0.977}\\
        \midrule %第二条线的规则
        FPS & \textcolor{red}{203.9} & 18.4 & 5.3 & 54.2 & \textcolor{blue}{54.9}\\
        \bottomrule %第三条线的规则
    \end{tabular}
\end{table}
```



## Figure

```latex
%插入图
\begin{figure}[h] %指定图片位置, “h” here, “t” top, “b” bottom, “p”
    \vspace{-0.5cm} %调整图片与上文的垂直距离, 不仅适用于figure。
    \setlength{\abovecaptionskip}{-0.2cm}   %调整图片标题与图片距离
    \setlength{\belowcaptionskip}{-1cm}   %调整图片标题与下文距离
    \begin{center} %使图片居中
    \includegraphics[width=80mm]{./img/a3} %指定图片路径和大小。
    \end{center}
    \caption{Success and precision plots for the 20 fast motion video sequences selected from OTB2015 and KITTI datasets. Success plots use mean AUC for ranking and precision plots use threshold = 20 for ranking.} %图片注释
\end{figure}
```



## Algorithm

```latex
%生成算法
\usepackage{algorithm}
\usepackage{algorithmic}
%
\begin{algorithm}[htb] %指定算法位置, “h” here, “t” top, “b” bottom, “p”
    \caption{Our tracker’s algorithm} %标题
    \label{alg:Framwork} %标签, 好像可有可无。
    \begin{algorithmic}[1] %这个1 表示算法每一行都显示行号。
        \REQUIRE ~~\\ %算法的输入参数：Input
            x: training image patch\\
            $ P_{old} $: previous frame position\\
            $ S_{old} $: previous target scale
        \ENSURE ~~\\ %算法的输出：Output
            $ P_{new} $: new position\\
            $ S_{new} $: new target scale
        \STATE\textbf{translation estimation}\\
        $ \bullet $ The response score is calculated according to Equation 2.\\
        $ \bullet $ Set $ P_{new}$ to the target position that maximizes the response.\\
        \STATE\textbf{Scale estimation}\\
        $ \bullet $ Calculate the prediction scale $ S_{new^{'}} $ according to $ S_{old} $ using Equation 7.\\
        $ \bullet $ Extract scale sample with scale $ S_{new^{'}} $ at $ P_{new}$ and calculate the response score.\\
        $ \bullet $ Set $ S_{new} $ to the target scale that maximizes the response.
        \STATE\textbf{Model update}\\
        $ \bullet $ Calculate the update interval using Equation 8.\\
        $ \bullet $ Update the model when the update interval is reached.
    \end{algorithmic}
\end{algorithm}
```



# Matlab

## ubuntu安装matlab

```bash
# 一键安装脚本
# https://zhuanlan.zhihu.com/p/394298249
```





## 服务器跑matlab代码注意事项

```bash
# 后台跑matlab代码时, 跟其他程序用nohup的方式不一样。https://blog.csdn.net/qq_15015187/article/details/107740427

nohup /usr/local/Matlab/2018a/bin/matlab -nodesktop -nodisplay -r pso > outfile.txt </dev/null &
3554 fudan703_2
```



# 数据集相关代码

## 图片信息统计

```python
import os
import numpy as np
import matplotlib.pyplot as plt

from PIL import Image

from docx import Document
from docx.shared import Pt
from docx.oxml.ns import qn
from docx.shared import Inches

# 1) 统计文件夹下图片的数量
def count_images_in_folder(folder_path):
    file_list = [f for f in os.listdir(folder_path) if os.path.isfile(os.path.join(folder_path, f))]
    image_count = sum(1 for file in file_list if file.lower().endswith(('.png', '.jpg', '.jpeg', '.gif', '.bmp')))
    return image_count

# 2) 统计图片后缀类型和数量
def count_image_extensions(folder_path):
    file_list = [f for f in os.listdir(folder_path) if os.path.isfile(os.path.join(folder_path, f))]
    extensions = [os.path.splitext(file.lower())[1] for file in file_list if file.lower().endswith(('.png', '.jpg', '.jpeg', '.gif', '.bmp'))]
    extension_counts = {ext: extensions.count(ext) for ext in set(extensions)}
    return extension_counts

# 3) 绘制所有图片的像素值分布直方图
def plot_pixel_distribution(folder_path):
    file_list = [f for f in os.listdir(folder_path) if os.path.isfile(os.path.join(folder_path, f))]
    pixel_values = []

    for file in file_list:
        if file.lower().endswith(('.png', '.jpg', '.jpeg', '.gif', '.bmp')):
            image = Image.open(os.path.join(folder_path, file))
            pixel_values.extend(np.array(image).flatten())

    pixel_values = np.array(pixel_values)
    hist, bins = np.histogram(pixel_values, bins=256, range=(0, 256), density=True)

    # 创建并保存直方图图像
    plt.figure(figsize=(10, 5))
    plt.plot(bins[:-1], hist, color='b', alpha=0.7)
    plt.xlabel('Pixel Value')
    plt.ylabel('Frequency')
    plt.title('Pixel Value Distribution')
    plt.grid(True)
    plt.savefig('pixel_distribution.png')  # 保存为文件
    plt.close()

    return hist, bins

# 4) 统计不同尺寸图片的数量和种类
def count_image_sizes(folder_path):
    file_list = [f for f in os.listdir(folder_path) if os.path.isfile(os.path.join(folder_path, f))]
    image_sizes = {}
    for file in file_list:
        if file.lower().endswith(('.png', '.jpg', '.jpeg', '.gif', '.bmp')):
            image = Image.open(os.path.join(folder_path, file))
            width, height = image.size
            size_tuple = (width, height)
            if size_tuple in image_sizes:
                image_sizes[size_tuple] += 1
            else:
                image_sizes[size_tuple] = 1
    return image_sizes

def create_document_with_font(folder_path):
    document = Document()

    # 设置中文字体（宋体）和西文字体（新罗马）
    style = document.styles['Normal']
    font = style.font
    font.name = 'Times New Roman'
    font.size = Pt(12)
    style._element.rPr.rFonts.set(qn('w:eastAsia'), '宋体')

    document.add_heading('数据集统计信息', 0)

    # 添加内容到文档
    # 1) 统计文件夹下图片的数量并添加到文档
    print('统计文件夹下图片的数量并添加到文档')
    image_count = count_images_in_folder(data_folder)
    document.add_heading('1) 文件夹下图片的数量', level=1)
    document.add_paragraph(f"共有 {image_count} 张图片。")

    # 2) 统计图片后缀类型和数量并添加到文档
    print('统计图片后缀类型和数量并添加到文档')
    extension_counts = count_image_extensions(data_folder)
    document.add_heading('2) 不同类型图片后缀及数量', level=1)
    for ext, count in extension_counts.items():
        document.add_paragraph(f"{ext}: {count} 张")


    # 3) 绘制所有图片的像素值分布直方图并添加到文档
    print('绘制所有图片的像素值分布直方图并添加到文档')
    hist, bins = plot_pixel_distribution(data_folder)
    document.add_heading('3) 图片像素值分布直方图', level=1)
    document.add_paragraph("详细信息见附图：")

    # 添加直方图图像到文档
    document.add_picture('pixel_distribution.png', width=Inches(6))

    # 4) 统计不同尺寸图片的数量和种类并添加到文档
    print('统计不同尺寸图片的数量和种类并添加到文档')
    image_sizes = count_image_sizes(data_folder)
    document.add_heading('4) 不同尺寸的图片数量和种类', level=1)
    for size, count in image_sizes.items():
        document.add_paragraph(f"尺寸 {size}: {count} 张图片")


    # 保存文档
    document.save('Image_Statistics_Report.docx')


# 指定数据集文件夹路径
data_folder = '/data/hongda/ImageInpainting_polyp/dataset/polyp/label'
create_document_with_font(data_folder)
```





## 数据集图片处理

```bash
#!/bin/bash

echo "选择要执行的操作："
echo "1) 修改图片尺寸"
echo "2) 将图片转换为二值图"
read -p "输入选择（1或2）: " choice

if [ "$choice" = "1" ]; then
read -p "输入图片路径: " origin_file_dir
read -p "输入目标宽度: " width
read -p "输入目标高度: " height
read -p "输入目标目录: " target_directory
python3 image_processor.py --resize --origin_directory "$origin_file_dir" --width "$width" --height "$height" --target_directory "$target_directory"
elif [ "$choice" = "2" ]; then
read -p "输入图片路径: " origin_file_dir
read -p "输入二值化阈值（0-255）: " threshold
read -p "输入目标目录: " target_directory
python3 image_processor.py --binarize --origin_directory "$origin_file_dir" --threshold "$threshold" --target_directory "$target_directory"
    else
    echo "无效选择"
    fi
```





## bash脚本对应的python代码

```python
import argparse
from PIL import Image
import shutil

import os
from natsort import natsorted

def get_sorted_image_paths(directory):
    """
    Reads all image file paths from the specified directory and sorts them naturally.

    :param directory: The directory to read image files from.
    :return: A list of sorted image file paths.
    """
    # Supported image formats
    image_extensions = {'.jpg', '.jpeg', '.png', '.gif', '.bmp', '.tiff', '.webp'}

    # Get all files in the directory
    all_files = os.listdir(directory)

    # Filter out image files
    image_files = [file for file in all_files if os.path.splitext(file)[1].lower() in image_extensions]

    # Sort using natsort
    sorted_image_files = natsorted(image_files)

    # Construct full file paths
    full_paths = [os.path.join(directory, file) for file in sorted_image_files]

    return full_paths
    
def resize_and_save_images(origin_directory, target_width, target_height, target_directory):
    if not os.path.exists(target_directory):
        os.makedirs(target_directory)

    image_paths = get_sorted_image_paths(origin_directory)

    for path in image_paths:
        try:
            with Image.open(path) as img:
                # Resize the image
                resized_img = img.resize((target_width, target_height), Image.ANTIALIAS)

                # Construct the new path
                base_name = os.path.basename(path)
                new_path = os.path.join(target_directory, base_name)
                
                # Save the resized image to the new path
                resized_img.save(new_path)
        except IOError:
            print(f"Unable to open or save image at path: {path}")


def convert_to_binary_and_save(origin_directory, threshold, target_directory):
    if not os.path.exists(target_directory):
        os.makedirs(target_directory)

    image_paths = get_sorted_image_paths(origin_directory)

    for path in image_paths:
        try:
            with Image.open(path) as img:
                # Convert to grayscale
                grayscale_img = img.convert('L')

                # Apply threshold
                binary_img = grayscale_img.point(lambda x: 255 if x > threshold else 0, '1')

                # Construct the new path
                base_name = os.path.basename(path)
                new_path = os.path.join(target_directory, base_name)
                
                # Save the binary image to the new path
                binary_img.save(new_path)
        except IOError:
            print(f"Unable to open or process image at path: {path}")

def rename_and_copy_files(origin_directory, target_directory):
    """
    Renames all files in the source folder to a six-digit number format, 
    starting from 000000, and copies them to the destination folder.

    :param src_folder: Path to the source folder containing the files.
    :param dest_folder: Path to the destination folder where files will be copied.
    """
    # Create the destination folder if it doesn't exist
    if not os.path.exists(target_directory):
        os.makedirs(target_directory)

    # Get a list of all files in the source directory, and sort
    files = get_sorted_image_paths(origin_directory)

    # Rename and copy each file
    for i, filename in enumerate(files):
        # Extract the file extension
        extension = os.path.splitext(filename)[1]

        # Create the new file name
        new_filename = f"{i+1:06d}{extension}"

        # Construct full file paths
        old_file = os.path.join(origin_directory, filename)
        new_file = os.path.join(target_directory, new_filename)

        # Copy and rename the file
        shutil.copy2(old_file, new_file)

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='图片处理脚本')
    parser.add_argument('--origin_directory', type=str, help='图片路径')
    parser.add_argument('--target_directory', type=str, help='图片路径')

    parser.add_argument('--resize', action='store_true', help='调整图片尺寸')
    parser.add_argument('--width', type=int, help='目标宽度', default=0)
    parser.add_argument('--height', type=int, help='目标高度', default=0)

    parser.add_argument('--binarize', action='store_true', help='将图片转换为二值图')
    parser.add_argument('--threshold', type=int, help='二值化阈值', default=128)

    parser.add_argument('--rename', action='store_true', help='将图片转换为二值图')


    args = parser.parse_args()

    if args.resize:
        resize_and_save_images(args.origin_directory, args.width, args.height, args.target_directory)
    elif args.binarize:
        convert_to_binary_and_save(args.origin_directory, args.threshold, args.target_directory)
    elif args.rename:
        rename_and_copy_files(args.origin_directory, args.target_directory)
```





# Final