# 脚本说明

来自官方的脚本，我修改ubuntu启动的bug，同时把一个默认的配置文件，放到 /usr/local/etc/xray/config.json

必备工具
```
apt-get update
apt -y install netcat zip
nc -zv 服务器IP地址 443
```


```
bash -c "$(curl -L https://github.com/shake/Xray-install/raw/main/install-release.sh)" @ install
```

测试的时候，我经常是重复安装和卸载

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ remove --purge
```

安装完成后，xray服务无法启动，是因为配置文件没有设置uuid和密钥

```
installed: /usr/local/bin/xray
installed: /usr/local/share/xray/geoip.dat
installed: /usr/local/share/xray/geosite.dat
installed: /usr/local/etc/xray/config.json
installed: /var/log/xray/
installed: /var/log/xray/access.log
installed: /var/log/xray/error.log
installed: /etc/systemd/system/xray.service
installed: /etc/systemd/system/xray@.service
removed: /tmp/tmp.wja2oGfDJ9
info: Xray v1.8.11 is installed.
You may need to execute a command to remove dependent software: apt purge curl unzip
Created symlink /etc/systemd/system/multi-user.target.wants/xray.service → /etc/systemd/system/xray.service.
warning: Failed to enable and start the Xray service
```

## 创建uuid和密钥

```
cd /usr/local/bin/ && ./xray uuid > uuid && ./xray x25519 > key
```

## 查看

```
cat /usr/local/bin/uuid && cat /usr/local/bin/key
```

## 修改配置文件

```
/usr/local/etc/xray/config.json
```

直接修改这个配置文件，更换上uuid和私钥就可以。就是2个地方修改就可以。

## 启动

```
systemctl restart xray.service
```

查看状态

```
systemctl status xray.service
```

服务正常，表示服务器端完成。

```
# systemctl status xray.service
● xray.service - Xray Service
     Loaded: loaded (/etc/systemd/system/xray.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/xray.service.d
             └─10-donot_touch_single_conf.conf
     Active: active (running) since Sun 2024-05-19 06:12:48 UTC; 10s ago
       Docs: https://github.com/xtls
   Main PID: 3699 (xray)
      Tasks: 7 (limit: 1030)
     Memory: 3.6M
        CPU: 54ms
     CGroup: /system.slice/xray.service
             └─3699 /usr/local/bin/xray run -config /usr/local/etc/xray/config.json

May 19 06:12:48 racknerd-3ab4502 systemd[1]: Started Xray Service.
May 19 06:12:48 racknerd-3ab4502 xray[3699]: Xray 1.8.11 (Xray, Penetrates Everything.) 45ab4cb (go1.22.>
May 19 06:12:48 racknerd-3ab4502 xray[3699]: A unified platform for anti-censorship.
May 19 06:12:48 racknerd-3ab4502 xray[3699]: 2024/05/19 06:12:48 [Info] infra/conf/serial: Reading confi>
May 19 06:12:48 racknerd-3ab4502 xray[3699]: 2024/05/19 06:12:48 [Warning] core: Xray 1.8.11 started
```

## BBR3

1G内存的虚拟机，就不建议装BBR3.

<details><summary>参考资料</summary>

手工方式无法搞定，只能借助脚本

```
curl -sS -O https://raw.githubusercontent.com/kejilion/sh/main/kejilion.sh && chmod +x kejilion.sh && ./kejilion.sh
```

选择13，进入二级页面，选择16升级BBR3. 看了脚本，也实在没理解手工操作的区别。


默认1G内存，不够，需要把swap分区扩大到1G。

[ubuntu官方调整swap分区文档](https://askubuntu.com/questions/178712/how-to-increase-swap-space?newreg=59195221d1)

查看当前swap分区，清除文件系统标识符

```
swap_partitions=$(grep -E '^/dev/' /proc/swaps | awk '{print $1}')
echo "$partition"
swapoff "$partition"
wipefs -a "$partition"
mkswap -f "$partition"
```
确认

```
swapon -s
```
关闭swap分区

```
swapoff -a
```
创建1Gswap 文件

```
dd if=/dev/zero of=/swapfile bs=1024 count=1M
```
设置

```
# Set the correct permissions
chmod 0600 /swapfile

mkswap /swapfile  # Set up a Linux swap area
swapon /swapfile  # Turn the swap on
```

查看是否正常工作

```
grep Swap /proc/meminfo
```

永久生效

```
echo "/swapfile swap swap defaults 0 0" >> /etc/fstab
```

优化swap性能

[How To Add Swap Space on Ubuntu 22.04
](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-22-04)

```
cat /proc/sys/vm/swappiness

sysctl vm.swappiness=10

cat /proc/sys/vm/vfs_cache_pressure

sysctl vm.vfs_cache_pressure=50
```

修改 /etc/sysctl.conf

```
vm.swappiness=10
vm.vfs_cache_pressure=50
```

添加repo密钥

```
wget -qO - https://dl.xanmod.org/archive.key | sudo gpg --dearmor -vo /usr/share/keyrings/xanmod-archive-keyring.gpg
```
添加repo

```
echo 'deb [signed-by=/usr/share/keyrings/xanmod-archive-keyring.gpg] http://deb.xanmod.org releases main' | sudo tee /etc/apt/sources.list.d/xanmod-release.list
```

安装新内核

```
sudo apt update && sudo apt install linux-xanmod-x64v3
```

重启

开启bbr3，/etc/sysctl.conf，添加两行

```
net.core.default_qdisc=fq_pie
net.ipv4.tcp_congestion_control=bbr

```

查看结果

```
sysctl -p
```

查看bbr3状态，已经无法使用这个命令检查bbr3，因为已经编译进去内核。

```
modinfo tcp_bbr
cat /proc/sys/net/ipv4/tcp_congestion_control
```
</details> 

## 客户端配置

| 名称        | 值                                          |
| :---------- | :------------------------------------------ |
| 地址        | IP 或服务端的域名                           |
| 端口        | 443      (服务器设置相同，可以其他端口）                                   |
| 用户ID      | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx (uuid，服务器相同）       |
| 流控        | xtls-rprx-vision                            |
| 传输协议    | tcp                                         |
| 传输层安全  | reality                                     |
| SNI         | www.amazon.com   (服务器设置相同）                      |
| Fingerprint | chrome                                      |
| PublicKey   | wC-8O2vI-7OmVq4TVNBA57V_g4tMDM7jRXkcBYGMYFw（公钥） |
| shortId     | 98     (服务器设置相同，，服务器端设置也可以为空）                       |
| spiderX     | /          （爬虫初始路径与参数，建议每个客户端不同，可以是/abc或者 /）                                 |


[官方客户端中文说明](https://github.com/XTLS/REALITY?tab=readme-ov-file)

### v2rayN

![v2rayN-reality](https://github.com/shake/Xray-install/blob/77d0aaf3c079cfe75d0a39e5ddd21967e945839e/image/reality-03.jpg)
![v2rayN-reality](https://github.com/shake/Xray-install/blob/77d0aaf3c079cfe75d0a39e5ddd21967e945839e/image/reality-04.jpg)


### Nekobox

使用**系统代理**，核心选择**sing-box**

默认路由设置

![route](https://github.com/shake/Xray-install/blob/a229eed7eec6e4e37978e1030096e11e2767b662/image/nekobox-02.jpg)

手工输入配置Reality节点

![Nekobox-reality](https://github.com/shake/Xray-install/blob/4f4840b599774712d7114a396214ed7eb27fd90e/image/nekobox-01.jpg)



<details><summary><font color=Blue>参考资料</font></summary>

     # Xray-install

Bash script for installing Xray in operating systems such as CentOS / Debian / OpenSUSE that support systemd.

[Filesystem Hierarchy Standard (FHS)](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

```
installed: /etc/systemd/system/xray.service
installed: /etc/systemd/system/xray@.service

installed: /usr/local/bin/xray
installed: /usr/local/etc/xray/*.json

installed: /usr/local/share/xray/geoip.dat
installed: /usr/local/share/xray/geosite.dat

installed: /var/log/xray/access.log
installed: /var/log/xray/error.log
```

Notice: Xray will NOT log to `/var/log/xray/*.log` by default. Configure `"log"` to specify log files.

## Basic Usage

**Install & Upgrade Xray-core and geodata with `User=nobody`, but will NOT overwrite `User` in existing service files**

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

**Update geoip.dat and geosite.dat only**

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install-geodata
```

**Remove Xray, except json and logs**

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ remove
```

## Advance

**Install & Upgrade Xray-core to a pre-release version**

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --beta
```

**Install & Upgrade Xray-core and geodata with `logrotate`, `$time` can be in the format of 12:34:56**

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --logrotate $time
```
```
installed: /etc/systemd/system/logrotate@.service
installed: /etc/systemd/system/logrotate@.timer

installed: /etc/logrotate.d/xray
```

**Install & Upgrade Xray-core and geodata with `User=root`, which will overwrite `User` in existing service files**

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root
```

**Install & Upgrade Xray-core without geodata**

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --without-geodata
```

**Remove Xray, include json and logs**

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ remove --purge
```

## More Usage

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ help
```

## Stargazers over time

[![Stargazers over time](https://starchart.cc/XTLS/Xray-install.svg)](https://starchart.cc/XTLS/Xray-install)

</details> 
