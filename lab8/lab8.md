# 实验任务

* 安装并使用cuckoo。任意找一个程序，在cuckoo中trace获取软件行为的基本数据。

> https://cuckoosandbox.org/

# 实验过程

1. 安装`cuckoo`依赖

```shell
sudo apt-get install git mongodb libffi-dev build-essential python-django python python-dev python-pip python-pil python-sqlalchemy python-bson python-dpkt python-jinja2 python-magic python-pymongo python-gridfs python-libvirt python-bottle python-pefile python-chardet tcpdump -y
```

2. 安装Tcpdump并确认安装无误：

```shell
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
getcap /usr/sbin/tcpdump 
```

<img src="pic\1.png" style="zoom:80%;" />

3. 安装Pydeep：

```shell
$ wget http://sourceforge.net/projects/ssdeep/files/ssdeep-2.13/ssdeep-2.13.tar.gz/download -O ssdeep-2.13.tar.gz
$ tar -zxf ssdeep-2.13.tar.gz
$ cd ssdeep-2.13
$ ./configure
$ make
$ sudo make install

#确认安装无误
$ ssdeep -V
Then proceed by installing pydeep:

$ sudo pip install pydeep
Validate that the package is installed:

$ pip show pydeep
---
Name: pydeep
Version: 0.2
Location: /usr/local/lib/python2.7/dist-packages
Requires:
```

4. 安装Volatility：

```shell
#先安装依赖
sudo pip3 install openpyxl
sudo pip3 install --user  ujson -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
#以下都利用换源来加速
sudo pip3 install --user  pycrypto -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
sudo pip3 install --user  distorm3 -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
sudo pip3 install --user  pytz -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com

#然后安装volatility
git clone https://github.com/volatilityfoundation/volatility.git
cd volatility
python setup.py build
sudo python setup.py install

#确认安装无误
python vol.py -h
```

<img src="pic\4.png" style="zoom:80%;" />

安装Volatility时遇到问题：

<img src="pic\3.png" style="zoom:80%;" />

改用这条命令，换源之后即可避免此类问题，还能加速(之后跟pip相关的下载指令都随之更改)

```shell
sudo pip3 install --user  ujson -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
```



5. 安装Cuckoo：

```shell
sudo apt install virtualenv
virtualenv venv
. venv/bin/activate
(venv)$ pip3 install -U pip setuptools
(venv)$ pip3 install -U cuckoo  # 安装cuckoo时出现错误，改用pip，不能用pip3
```

<img src="pic\5.png" style="zoom:80%;" />

改进后

<img src="pic\6.png" style="zoom:80%;" />

6. win7中：安装python环境和PIL,Pillow等，关闭防火墙、自动更新等

<img src="pic\7.png" style="zoom:60%;" />

<img src="pic\8.png" style="zoom:60%;" />

- 在virtualbox中位win7设置一块hostonly网卡

<img src="pic\9.png" style="zoom:60%;" />

<img src="pic\10.png" style="zoom:60%;" />

* 在win 7配置ip网络

<img src="pic\11.png" style="zoom:60%;" />

- 检查是否配置成功，主机和客机是否能通信。

  主机中操作

  ```shell
  ping 192.168.56.101
  ```

  客机中操作

  ```shell
  ping 192.168.56.1
  ```

- 设置IP报文转发，这是在Ubuntu中的操作，因为win7无法上网，所以要通过主机才能访问网络，所以需要以下操作; 流量转发服务：

  ```shell
  sudo vim /etc/sysctl.conf
  net.ipv4.ip_forward=1
  sudo sysctl -p /etc/systl.conf
  ```

<img src="pic\12.png" style="zoom:60%;" />

<img src="pic\13.png" style="zoom:80%;" />

<img src="pic\14.png" style="zoom:80%;" />

7. 使用iptables提供NAT机制

```shell
$ sudo iptables -A FORWARD -o eth0 -i vboxnet0 -s 192.168.56.0/24 -m conntrack --ctstate NEW -j ACCEPT
$ sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
$ sudo iptables -A POSTROUTING -t nat -j MASQUERADE
$ sudo vim /etc/network/interfaces
# 新增下列兩行
pre-up iptables-restore < /etc/iptables.rules #开机自启动规则
post-down iptables-save > /etc/iptables.rules #保存规则
sudo apt-get install iptables=persistent
sudo netfilter-persistent save
#dns
$ sudo apt-get install -y dnsmasq
$ sudo service dnsmasq start
```

- 设置cuckoo配置文件，配置virtualbox.conf

  ```shell
  $ vim virtualbox.conf
  machines = cuckoo1 # 指定VirtualBox中Geust OS的虛擬機名稱
  [cuckoo1] # 對應machines
  label = cuckoo1  .
  platform = windows
  ip = 192.168.56.101 # 指定VirtualBox中Geust OS的IP位置
  snapshot =snapshot
  #配置reporting.conf
  $ vim reporting.conf
  [jsondump]
  enabled = yes # no -> yes
  indent = 4
  calls = yes
  [singlefile]
  # Enable creation of report.html and/or report.pdf?
  enabled = yes # no -> yes
  # Enable creation of report.html?
  html = yes # no -> yes
  # Enable creation of report.pdf?
  pdf = yes # no -> yes
  [mongodb]
  enabled = yes # no -> yes
  host = 127.0.0.1
  port = 27017
  db = cuckoo
  store_memdump = yes 
  paginate = 100
  #配置cuckoo.conf
  version_check = no
  machinery = virtualbox
  memory_dump = yes
  [resultserver]
  ip = 192.168.56.1
  port = 2042
  ```

- 启动cuckoo,进入venv中，输入命令启动cuckoo服务；另外开出一个控制台，启动cuckoo web服务

  ```shell
  cuckoo
  cuckoo web runserver
  ```

- 用浏览器打开所给网址：

<img src="pic\15.png" style="zoom:60%;" />





# 实验参考

> Could not find a version that satisfies the requirement jupyter (from versions: )错误:https://blog.csdn.net/u013983033/article/details/93120180
>
> win7设置IP地址：https://jingyan.baidu.com/article/b24f6c82c4d2d586bee5da6b.html
>
> 谌雯馨同学的仓库：https://github.com/chencwx/Software-and-system-security/tree/master/恶意软件防御体系