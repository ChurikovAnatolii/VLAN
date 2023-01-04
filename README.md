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
**Вывод: При отключении интерфейса eth1 на роутере inetRouter пинги перестали идти по bond-иньерфейсу до тех пор пока eth1 не был поднят. Задание выполнено.**
<br>

## Vlan для testClient и testServer


