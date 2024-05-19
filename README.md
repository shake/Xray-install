# 脚本说明

来自官方的脚本，我修改ubuntu启动的bug，同时把一个默认的配置文件，放到 /usr/local/etc/xray/config.json

bash -c "$(curl -L https://github.com/shake/Xray-install/raw/main/install-release.sh)" @ install

你会发现无法启动，是因为配置文件没有设置uuid和密钥

## 创建uuid和密钥

cd /usr/local/bin/

./xray uuid > uuid

./xray x25519 > key

## 查看

cat uuid

cat key

# /usr/local/etc/xray/config.json

直接修改这个配置文件，更换上uuid和私钥就可以。

## 启动

systemctl restart xray.service

systemctl status xray.service

服务正常，表示服务器端完成。

## 客户端配置

| 名称        | 值                                          |
| :---------- | :------------------------------------------ |
| 地址        | IP 或服务端的域名                           |
| 端口        | 443      (服务器设置相同，可以其他端口）                                   |
| 用户ID      | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx (服务器设置相同）       |
| 流控        | xtls-rprx-vision                            |
| 传输协议    | tcp                                         |
| 传输层安全  | reality                                     |
| SNI         | learn.microsoft.com   (服务器设置相同）                      |
| Fingerprint | chrome                                      |
| PublicKey   | wC-8O2vI-7OmVq4TVNBA57V_g4tMDM7jRXkcBYGMYFw（这是公钥） |
| shortId     | 6ba85179e30d4fc2     (服务器设置相同）                       |
| spiderX     | /          （爬虫初始路径与参数，建议每个客户端不同，可以是/abc或者 /）                                 |



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
