# 实验内容

* 搜集市面上主要的路由器厂家、在厂家的官网中寻找可下载的固件在CVE漏洞数据中查找主要的家用路由器厂家的已经公开的漏洞，选择一两个能下载到切有已经公开漏洞的固件。如果能下载对应版本的固件，在QEMU中模拟运行。确定攻击面（对哪个端口那个协议进行Fuzzing测试），尽可能多的抓取攻击面正常的数据包（wireshark）。
* 查阅BooFuzz的文档，编写这对这个攻击面，这个协议的脚本，进行Fuzzing。配置BooFuzz QEMU的崩溃异常检测，争取触发一次固件崩溃，获得崩溃相关的输入测试样本和日志。尝试使用调试器和IDA-pro监视目标程序的崩溃过程，分析原理。

# 实验过程

1. 克隆 Firmware Analysis Toolkit 工具集仓库


```shell
# 1. 安装依赖
sudo apt-get install busybox-static fakeroot git dmsetup kpartx netcat-openbsd nmap python-psycopg2 python3-psycopg2 snmp uml-utilities util-linux vlan

# 2. clone 
git clone --recursive https://github.com/attify/firmware-analysis-toolkit.git
```

2. 安装binwalk


```shell
# 1. 安装依赖和binwalk
sudo apt install binwalk
# 2. 对于 python2.x，还需要安装以下的库
sudo -H pip install git+https://github.com/ahupp/python-magic
sudo -H pip install git+https://github.com/sviehb/jefferson
```


- 测试是否安装成功：

```shell
binwalk
```

<img src="pic\1.png" style="zoom:80%;" />

3. 安装Firmadyne

* 进入Firmadyne目录，然后打开firmadyne.config，修改 FIRMWARE_DIR的路径为当前Firmadyne目录的绝对路径

```shell
git clone https://github.com/firmadyne/firmadyne
cd firmadyne
vim firmadyne.config
```

<img src="pic\2.png" style="zoom:80%;" />

* 安装Firmadyne【时间未免有一点太长了吧……】【截图是无限等待中的某一刻】

```shell
sh ./download.sh
sudo ./setup.sh 
```

<img src="pic\3.png" style="zoom:80%;" />