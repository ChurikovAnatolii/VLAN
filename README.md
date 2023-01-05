# VLAN

## Bond между inetRouter и centralRouter
<br>

***- Для запуска стенда используется команда vagrant up && ansible-playbook vlan.yml.***

***- Для реализации bond интерфейсов между роутерами был использован модуль nmcli в ansible.***

***- После запуска стенда проверим, что интерфейсы bond созданы на обоих роутерах и они в подняты.***

```console
[vagrant@centralRouter ~]$ sudo cat /proc/net/bonding/bond0 
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: load balancing (round-robin)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:81:fd:9b
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:6a:cf:da
Slave queue ID: 0

[vagrant@inetRouter ~]$ sudo cat /proc/net/bonding/bond0 
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: load balancing (round-robin)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 2
Permanent HW addr: 08:00:27:82:41:c6
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:e5:45:75
Slave queue ID: 0

```

**Агрегирование портов было сделано с типом round robin, поэтому при отлючении одного интерфеса ожидаем, что сетевые пакеты ходить перестанут. Проверим это запустив пинг на centralRouter в сторону inetRouter, а затем отключим интерфейс eth1 на inetRouter.**

```console 
[vagrant@centralRouter ~]$ ping 192.168.255.1
PING 192.168.255.1 (192.168.255.1) 56(84) bytes of data.
64 bytes from 192.168.255.1: icmp_seq=1 ttl=64 time=0.989 ms
64 bytes from 192.168.255.1: icmp_seq=2 ttl=64 time=0.845 ms
64 bytes from 192.168.255.1: icmp_seq=3 ttl=64 time=1.06 ms
64 bytes from 192.168.255.1: icmp_seq=4 ttl=64 time=0.905 ms
64 bytes from 192.168.255.1: icmp_seq=5 ttl=64 time=0.818 ms
64 bytes from 192.168.255.1: icmp_seq=6 ttl=64 time=0.844 ms
64 bytes from 192.168.255.1: icmp_seq=7 ttl=64 time=0.928 ms
64 bytes from 192.168.255.1: icmp_seq=8 ttl=64 time=1.06 ms
64 bytes from 192.168.255.1: icmp_seq=9 ttl=64 time=0.888 ms
64 bytes from 192.168.255.1: icmp_seq=10 ttl=64 time=0.844 ms
64 bytes from 192.168.255.1: icmp_seq=11 ttl=64 time=0.977 ms
64 bytes from 192.168.255.1: icmp_seq=12 ttl=64 time=0.850 ms
64 bytes from 192.168.255.1: icmp_seq=13 ttl=64 time=0.992 ms
64 bytes from 192.168.255.1: icmp_seq=14 ttl=64 time=0.904 ms
64 bytes from 192.168.255.1: icmp_seq=15 ttl=64 time=0.885 ms
64 bytes from 192.168.255.1: icmp_seq=16 ttl=64 time=1.02 ms

[vagrant@inetRouter ~]$ sudo ip link set down eth1

[vagrant@inetRouter ~]$ sudo ip link set up eth1

--- 192.168.255.1 ping statistics ---
74 packets transmitted, 74 received, 0% packet loss, time 73066ms
rtt min/avg/max/mdev = 0.735/14140.117/45250.748/15208.811 ms, pipe 46
[vagrant@centralRouter ~]$ 

```
---
**Вывод: При отключении интерфейса eth1 на роутере inetRouter пинги перестали идти по bond-интерфейсу, до тех пор пока eth1 не был поднят. Задание выполнено.**
<br>

## Vlan для testClient и testServer
<br>


***- Для вполнения задания по вланам в вагрант файл были добавлены хосты testClient1, testClien2, testServer1, testServer2. Для создания vlan был использован модуль nmcli в ansible.***

***- После запуска стенда зайдем на хосты и проверим, xnj vlan  созданы и в апе. Пропингуем клиентов с серверов и посмотрим в tcpdump, что пакеты приходят с тэгом vlan 100 и 101.***

***- Cначала для связки testClient1, testServer1***
```console

4: eth1.100@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:42:46:f4 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 brd 10.10.10.255 scope global noprefixroute eth1.100
       valid_lft forever preferred_lft forever
    inet6 fe80::3ee2:2d41:ba81:4c8c/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
[vagrant@testClient1 ~]$ 

[vagrant@testClient1 ~]$ sudo tcpdump -nvvv -e -i eth1 icmp -c 10
tcpdump: listening on eth1, link-type EN10MB (Ethernet), capture size 262144 bytes
17:26:48.416679 08:00:27:20:03:f0 > 08:00:27:42:46:f4, ethertype 802.1Q (0x8100), length 102: vlan 100, p 0, ethertype IPv4, (tos 0x0, ttl 64, id 50800, offset 0, flags [DF], proto ICMP (1), length 84)
    10.10.10.254 > 10.10.10.1: ICMP echo request, id 26909, seq 14, length 64
17:26:48.416746 08:00:27:42:46:f4 > 08:00:27:20:03:f0, ethertype 802.1Q (0x8100), length 102: vlan 100, p 0, ethertype IPv4, (tos 0x0, ttl 64, id 28773, offset 0, flags [none], proto ICMP (1), length 84)
    10.10.10.1 > 10.10.10.254: ICMP echo reply, id 26909, seq 14, length 64
17:26:49.418141 08:00:27:20:03:f0 > 08:00:27:42:46:f4, ethertype 802.1Q (0x8100), length 102: vlan 100, p 0, ethertype IPv4, (tos 0x0, ttl 64, id 51799, offset 0, flags [DF], proto ICMP (1), length 84)

```

***- А теперь для testClient2, testServer2***

```console
4: eth1.101@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:35:11:37 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 brd 10.10.10.255 scope global noprefixroute eth1.101
       valid_lft forever preferred_lft forever
    inet6 fe80::8a1d:2c06:9b35:870/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
[vagrant@testClient2 ~]$ 

[vagrant@testClient2 ~]$ sudo tcpdump -nvvv -e -i eth1 icmp
tcpdump: listening on eth1, link-type EN10MB (Ethernet), capture size 262144 bytes
17:32:54.898592 08:00:27:41:55:0a > 08:00:27:35:11:37, ethertype 802.1Q (0x8100), length 102: vlan 101, p 0, ethertype IPv4, (tos 0x0, ttl 64, id 47097, offset 0, flags [DF], proto ICMP (1), length 84)
    10.10.10.254 > 10.10.10.1: ICMP echo request, id 26890, seq 1, length 64
17:32:54.898694 08:00:27:35:11:37 > 08:00:27:41:55:0a, ethertype 802.1Q (0x8100), length 102: vlan 101, p 0, ethertype IPv4, (tos 0x0, ttl 64, id 24520, offset 0, flags [none], proto ICMP (1), length 84)
    10.10.10.1 > 10.10.10.254: ICMP echo reply, id 26890, seq 1, length 64
17:32:55.900030 08:00:27:41:55:0a > 08:00:27:35:11:37, ethertype 802.1Q (0x8100), length 102: vlan 101, p 0, ethertype IPv4, (tos 0x0, ttl 64, id 47852, offset 0, flags [DF], proto ICMP (1), length 84)
```
---
**Вывод: интерфейсы vlan настроены на хостах в соответствии с заданием, в дампе видно, что пинги приходят с тэгом vlan 100 и 101. Задание выполнено.**

