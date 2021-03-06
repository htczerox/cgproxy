# Transparent Proxy with cgroup v2



## Introduction

cgproxy will **transparent** proxy anything running in specific cgroup. It resembles with *proxychains* and *tsock*, but without their disadvantages.


<!--ts-->
   * [Transparent Proxy with cgroup v2](#transparent-proxy-with-cgroup-v2)
      * [Introduction](#introduction)
      * [Prerequest](#prerequest)
      * [How to install](#how-to-install)
      * [How to use](#how-to-use)
      * [Global transparent proxy](#global-transparent-proxy)
      * [Other useful tools provided in this project](#other-useful-tools-provided-in-this-project)
      * [NOTES](#notes)
      * [TIPS](#tips)
      * [Licences](#licences)

<!-- Added by: fancy, at: Thu 23 Apr 2020 01:23:57 PM HKT -->

<!--te-->


## Prerequest

- cgroup2

  For now, linux default using cgroup v1 for compatibility, this project need cgroup v2, you need disable cgroup v1 and enable cgroup v2 in your system.  see [Disabling v1 cgroups](https://wiki.archlinux.org/index.php/Cgroups#Disabling_v1_cgroups).

- TPROXY

  A process listening on port (e.g.  12345)  to accept iptables TPROXY, for example v2ray's dokodemo-door  in tproxy mode.

## How to install

```bash
mkdir build && cd build && cmake .. && make && make install
```

It is alreay in [archlinux AUR](https://aur.archlinux.org/packages/cgproxy/).

## How to use

- First enable service

  ```bash
  sudo systemctl enable --now cgproxy.service
  sudo systemctl status cgproxy.service
  ```

- Then prefix with cgproxy with your command, just like proxychains

  ```
  cgproxy <CMD>
  ```

- For example, test proxy

  ```bash
  cgproxy curl -vIs https://www.google.com
  ```

More config in `/etc/cgproxy.conf`:

```bash
## any process in this cgroup will be proxied
## must start with slash '/'
proxy_cgroup="/proxy.slice"
# proxy_cgroup="/user.slice"

## listening port of another proxy process, for example v2ray 
port=12345

## if you set to false, it's traffic won't go through proxy, but still can go direct to internet
enable_tcp=true
enable_udp=true
enable_ipv4=true
enable_ipv6=true

## v2ray outbound mark, depend on your v2ray setting
## only useful if v2ray process is also in proxy_cgroup, for example, you want to proxy whole userspace,
## and v2ray is also running in the same userspace
## otherwise ignore this
v2ray_outbound_mark=0xff # 255

## do not modify this if you don't known what you are doing
table=100
mark=0x01
mark_newin=0x02
```

If you changed config, remember to restart service

```bash
sudo systemctl restart cgproxy.service
```

## Global transparent proxy

- First, set `proxy_cgroup=/user.slice` in `/etc/cgproxy.conf`, this will proxy your whole user space

- Then, allow proxy software itself connect direct to internet, two available solutions:

  - Sloution 1: set all outbound mark in v2ray, and set `v2ray_outbound_mark` in `/etc/cgproxy.conf`

  - Sloution 2: run your proxy software in another cgroup that won't be proxyied

    ```bash
    # qv2ray as example
    run_in_cgroup --cgroup=/noproxy.slice qv2ray
    # v2ray as example
    run_in_cgroup --cgroup=/noproxy.slice v2ray --config config_file
    ```

- Finally, restart service `sudo systemctl restart cgproxy.service`, that's all

## 

## Other useful tools provided in this project

- `cgattach` attach specific process pid to specific cgroup which will create if not exist , cgroup can be only one level down exist cgroup, otherwise created fail.

  ```bash
  cgattch <pid> <cgroup>
  # example
  cgattch 9999 /proxy.slice
  ```

- `run_in_cgroup` run command in specific cgroup which will create if not exist , cgroup can be only one level down exist cgroup, otherwise created fail.

  ```bash
  run_in_cgroup --cgroup=CGROUP <COMMAND>
  # example
  run_in_cgroup --cgroup=/mycgroup.slice ping 127.0.0.1
  ```

## NOTES

- `cgattach` attach pid to specific cgroup, and has *suid* bit set by default, be careful to use on multi-user server for securiry. To avoid this situation,  you can remove the *suid* bit , then it will fallback to use *sudo*, with *visudo* you can restrict permission or set NOPASSWD for youself.
- TPROXY need root or cap_net_admin capability whatever process is listening on port,
  v2ray as example: sudo setcap cap_net_admin+ep /usr/lib/v2ray/v2ray

## TIPS

- `systemd-cgls` to see the cgroup hierarchical tree.

## Licences

cgproxy is licenced under [![License: GPL v3](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://www.gnu.org/licenses/gpl-2.0) 
