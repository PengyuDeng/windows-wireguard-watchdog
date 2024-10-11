# Windows-Wireguard-Watchdog

> 背景：Wireguard服务端是动态IP，Wireguard客户端配置的服务端地址是域名 Endpoint = xx.xxx.com:51820，写的DDNS脚本检测到IP变更会自动更新域名解析。每次宽带重拨IP就会变更，由于客户端重连使用的不是域名而是第一次连接时解析的域名对应的IP地址，导致一直连接不上服务端。

使用此PowerShell脚本，可以让你的Wireguard保持稳定的连接，服务端ip变更客户端仍可自动重新连接。

## 脚本流程

脚本主要执行了一下的步骤:

1. 解析配置文件读取连接服务端的域名。
2. 查询DNS服务器，获取A记录的IP地址。
3. 循环获取查询DNS，如果IP变更，把原配置文件的Endpoint替换成新的IP，生成新的配置文件，然后重新启动Wireguard服务。
4. 如果查询DNS服务器失3次，就停止Wireguard服务（防止因为Wireguard连接不上服务端导致查询DNS失败），然后再次执行第3步。

为什么要用新的IP生成配置文件？

Wireguard不会用我们配置的DNS获取IP，我们通过配置的DNS检测到IP变更，但是Wireguard解析域名可能检测不到变化，试过清除本地DNSClient缓存，不起作用，重启Wireguard服务他还是用原来的IP，导致连接失败。

## 安装步骤

让脚本以Windows服务方式运行 :

1. 把Windows-Wireguard-Watchdog放在C盘根路径下

2. 把Wireguard配置文件名改为my.conf

3. 用powershell终端管理员执行

  ```shell
  C:\windows-wireguard-watchdog\nssm.exe install MyWireGuardService "powershell.exe" "-ExecutionPolicy Bypass -File C:\windows-wireguard-watchdog\keep_wireguard_alive.ps1"
  ```

4. 启动服务
  ```shell
  C:\windows-wireguard-watchdog\nssm.exe start MyWireGuardService
  ```
  现在脚本就作为Windows服务运行了，电脑重启开机Wiregurd会自动连接。你可以打开Wireguard的UI查看日志，任务管理器查看Wireguard的进程。


## 关闭脚本服务

停止服务

```shell
C:\windows-wireguard-watchdog\nssm.exe stop MyWireGuardService
```

移除服务

```shell
C:\windows-wireguard-watchdog\nssm.exe remove MyWireGuardService
```

## 停止Wireguard服务

```shell
wireguard /uninstalltunnelservice mytemp
```
