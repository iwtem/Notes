## Linux 常用命令

### 1. 查看端口号

正在侦听的所有 TCP 或 UDP 端口，包括使用端口和套接字状态的服务：

```bash
netstat -tunlp
```

查询指定端口通过 grep 过滤：

```bash
netstat -tnlp | grep :端口号
```

### 2. 查看防火墙

查看防火墙状态

```bash
systemctl status firewalld
```

打开防火墙

```bash
systemctl start firewalld
```

### 3. Linux 开放端口

查看所有临时的开放端口

```bash
firewall-cmd --list-ports
```

查看所有永久的开放端口

```bash
firewall-cmd --list-ports --permanent
```

添加临时开放的端口

```bash
firewall-cmd --add-port=端口号/tcp
```

添加永久开放的端口

```bash
firewall-cmd --add-port=端口号/tcp --permanent
```

关闭临时端口

```bash
firewall-cmd --remove-port=端口号/tcp
```

关闭永久端口

```bash
firewall-cmd --remove-port=端口号/tcp --permanent
```

重启防火墙（配置后需重启）

```bash
firewall-cmd --reload

systemtl restart firewalld
```

