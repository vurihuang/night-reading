# CentOS Configure Proxy

<!-- TOC -->

- [CentOS Configure Proxy](#centos-configure-proxy)
    - [Install Shadowsocks Client and Configure it](#install-shadowsocks-client-and-configure-it)
    - [Service Auto Restart](#service-auto-restart)
    - [Check is Config works](#check-is-config-works)
    - [Install Privoxy and Configure it](#install-privoxy-and-configure-it)
    - [Apply](#apply)

<!-- /TOC -->

## Install Shadowsocks Client and Configure it

```
$ pip install shadowsocks
$ mkdir /etc/shadowsocks
```

Create `/etc/shadowsocks/shadowsocks.json`:

``` json
{
  "server": "ip",
  "server_port": port,
  "local_address": "127.0.0.1",
  "local_port": 1080,
  "password": "password",
  "timeout": 300,
  "method": "aes-256-cfb",
  "fase_open": false,
  "workers": 1
}
```

## Service Auto Restart

Create `/etc/systemd/system/shadowsocks.service`:

```
[Unit]
Description=Shadowsocks
[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks/shadowsocks.json
[Install]
WantedBy=multi-user.target
```

Start service:

```
$ systemctl enable shadowsocks.service
$ systemctl start shadowsocks.service
```

## Check is Config works

```
$ curl --socks5 127.0.0.1:1080 -I http://www.google.com
```

## Install Privoxy and Configure it

```
$ yum install -y privoxy
```

Edit `/etc/privoxy/config`:

```
listen-address 127.0.0.1:8118 # keep this as default
forward-socks5t / 127.0.0.1:1080 . # modify this to socks port, notice the last dot
```

## Apply

```
export http_proxy=socks5://127.0.0.1:1080
$ http_proxy=$http_proxy go get xxx
```



