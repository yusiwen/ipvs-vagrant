# ipvs-vagrant

this is a test ipvs(NAT) Vagrantfile.

## vm

| vm  | role         | ip                                                     |
|-----|--------------|--------------------------------------------------------|
| lb0 | LVS director | 192.168.66.11(DIP)、192.168.55.11（external interface) |
| lb1 | LVS director | 192.168.66.12(DIP)、192.168.55.12（external interface) |
| -   | VIP          | 192.168.55.100                                         |
| rs0 | RealServer   | 192.168.66.111(RIP)                                    |
| rs1 | RealServer   | 192.168.66.112(RIP)                                    |
| cl0 | Client       | 192.168.55.200                                         |

## start

```bash
vagrant up lb0 lb1 && vagrant reload lb0 lb1

vagrant up rs0 rs1

vagrant up cl0
```

### lb0 interface

```bash
vagrant@lb0:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:81:82:93:e5:d9 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 58120sec preferred_lft 58120sec
    inet6 fe80::81:82ff:fe93:e5d9/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:fc:74:d9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.66.11/24 brd 192.168.66.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet 192.168.55.100/32 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fefc:74d9/64 scope link 
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:0f:c3:81 brd ff:ff:ff:ff:ff:ff
    inet 192.168.55.11/24 brd 192.168.55.255 scope global enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe0f:c381/64 scope link 
       valid_lft forever preferred_lft forever
```

```bash
root@lb0:~# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.55.100:80 wrr
  -> 192.168.66.111:80            Masq    100    0          0         
  -> 192.168.66.112:80            Masq    100    0          0         

```

### rs0 interface

```bash
vagrant@rs0:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:56:36:6f:95:58 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 83646sec preferred_lft 83646sec
    inet6 fe80::56:36ff:fe6f:9558/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:05:30:c1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.66.111/24 brd 192.168.66.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe05:30c1/64 scope link 
       valid_lft forever preferred_lft forever
```

```bash
vagrant@rs0:~$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.66.11   0.0.0.0         UG    0      0        0 enp0s8
0.0.0.0         10.0.2.2        0.0.0.0         UG    100    0        0 enp0s3
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 enp0s3
10.0.2.2        0.0.0.0         255.255.255.255 UH    100    0        0 enp0s3
192.168.66.0    0.0.0.0         255.255.255.0   U     0      0        0 enp0s8
```

## ref

- [LVS+Iptables实现FULLNAT及原理分析](https://blog.dianduidian.com/post/lvs-snat%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90/)
- [LVS-NAT 原理介绍和配置实践](https://wsgzao.github.io/post/lvs-nat/)