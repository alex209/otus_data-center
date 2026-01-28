# Лабораторная №6

## VxLAN. EVPN L3

### Цель задания

Настроить маршрутизацию в рамках Overlay между клиентами.

### Задачи

1. Настроите каждого клиента в своем VNI
2. Настроите маршрутизацию между клиентами.
3. Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств

### Топология сети

![Топология сети](img/lab_06.png)

### Схема адресов IPv4

> **Примечание:** В текущей работе изменена адресация на линках Spine-Leaf с сетевой маски /30 на маску /31.

План адресов IPv4 для линков Spline-Leaf составлен по схеме `10.a.b.c/31`, где

- a - номер DC/POD,
- b - номер Spine,
- c - по очереди для подсети /31.

Адреса loopback `192.168.x.y/32`, где

- x - 1 для Spine, 0 - для Leaf,
- y - номер spine или leaf по порядку

Адреса для клиентов - `172.16.x.y/24`, где

- x - номер VLAN,
- y - порядковый адрес хоста `+10`

Адреса для SVI интерфейсов для диагностики - `172.16.x.y/24`, где

- x - номер VLAN,
- y - порядковый адрес хоста

Адресс шлюза по умолчанию `172.16.x.254`, где

- x - номер VLAN,

---

### Схема адресов IPv6

План адресов IPv6 для линков Spline-Leaf составлен по схеме `fdff:w:x:y:z::/127`, где

- w - номер DC,
- x - номер POD,
- y - номер Spine
- z - номер Leaf

Адреса loopback `fdfd:w:x:y:z::/128`, где

- w - номер DC,
- x - номер POD,
- y - 1 для Spine, 0 - для Leaf,
- z - номер Spine или Leaf по порядку

Адреса для клиентов - `fd:x:y::z/64`, где

- x - номер DC,
- y - номер VLAN
- z - порядковый адрес хоста `+10`

Адреса для SVI интерфейсов - `fd:x:y::z/64`, где

- x - номер DC,
- y - номер VLAN
- z - порядковый адрес хоста

Адресс шлюза по умолчанию `fd:x:y::ff`, где

- x - номер DC,
- y - номер VLAN

#### Итоговая таблица адресов Spine & Leaf

| Device | Interface | IP Address       | iPv6 address          |
| ------ | --------- | ---------------- | --------------------- |
| S01    | Lo0       | `192.168.1.1/32` | `fdfd:1:1:1:1::/128`  |
|        | Eth1      | `10.1.1.0/31`    | `fdff:1:1:1:1::/127`  |
|        | Eth2      | `10.1.1.2/31`    | `fdff:1:1:1:2::/127`  |
|        | Eth3      | `10.1.1.4/31`    | `fdff:1:1:1:3::/127`  |
|        | Eth4      | `10.1.1.6/31`    | `fdff:1:1:1:4::/127`  |
| ------ | --------- | --------------   | ------------------    |
| S02    | Lo0       | `192.168.1.2/32` | `fdfd:1:1:1:2::/128`  |
|        | Eth1      | `10.1.2.0/31`    | `fdff:1:1:2:1::/127`  |
|        | Eth2      | `10.1.2.2/31`    | `fdff:1:1:2:2::/127`  |
|        | Eth3      | `10.1.2.4/31`    | `fdff:1:1:2:3::/127`  |
|        | Eth4      | `10.1.2.6/31`    | `fdff:1:1:2:4::/127`  |
| ------ | --------- | --------------   | ------------------    |
| L01    | Lo0       | `192.168.0.1/32` | `fdfd:1:1:0:1::/128`  |
|        | Eth1      | `10.1.1.1/31`    | `fdff:1:1:1:1::1/127` |
|        | Eth2      | `10.1.2.1/31`    | `fdff:1:1:2:1::1/127` |
|        | vlan100   | `172.16.100.1`   | `fd:1:100::1/64`      |
|        | vlan200   | `172.16.200.1`   | `fd:1:200::1/64`      |
| ------ | --------- | --------------   | ------------------    |
| L02    | Lo0       | `192.168.0.2/32` | `fdfd:1:1:0:2::/128`  |
|        | Eth1      | `10.1.1.3/31`    | `fdff:1:1:1:2::1/127` |
|        | Eth2      | `10.1.2.3/31`    | `fdff:1:1:2:2::1/127` |
|        | vlan100   | `172.16.100.2`   | `fd:1:100::2/64`      |
|        | vlan200   | `172.16.200.2`   | `fd:1:200::2/64`      |
| ------ | --------- | --------------   | ------------------    |
| L03    | Lo0       | `192.168.0.3/32` | `fdfd:1:1:0:3::/128`  |
|        | Eth1      | `10.1.1.5/31`    | `fdff:1:1:1:3::1/127` |
|        | Eth2      | `10.1.2.5/31`    | `fdff:1:1:2:3::1/127` |
|        | vlan100   | `172.16.100.3`   | `fd:1:100::3/64`      |
|        | vlan200   | `172.16.200.3`   | `fd:1:200::3/64`      |
| ------ | --------- | --------------   | ------------------    |
| L04    | Lo0       | `192.168.0.4/32` | `fdfd:1:1:0:4::/128`  |
|        | Eth1      | `10.1.1.7/31`    | `fdff:1:1:1:4::1/127` |
|        | Eth2      | `10.1.2.7/31`    | `fdff:1:1:2:4::1/127` |
|        | vlan100   | `172.16.100.4`   | `fd:1:100::4/64`      |
|        | vlan200   | `172.16.200.4`   | `fd:1:200::4/64`      |

#### Итоговая таблица адресов клиентов

| Device  | Interface | VLAN  | IP Address         | IP Gateway       | IPv6 Address      | IPv6 Gateway   |
| ------- | --------- | ----- | ------------------ | ---------------- | ----------------- | -------------- |
| Linux_1 | eth0      | `100` | `172.16.100.11/24` | `172.16.100.254` | `fd:1:100::11/64` | `fd:1:100::ff` |
| Linux_2 | eth0      | `100` | `172.16.100.12/24` | `172.16.100.254` | `fd:1:100::12/64` | `fd:1:100::ff` |
| Linux_3 | eth0      | `100` | `172.16.100.13/24` | `172.16.100.254` | `fd:1:100::13/64` | `fd:1:100::ff` |
| Linux_4 | eth0      | `100` | `172.16.100.14/24` | `172.16.100.254` | `fd:1:100::14/64` | `fd:1:100::ff` |
| Linux_5 | eth0      | `200` | `172.16.200.15/24` | `172.16.200.254` | `fd:1:200::15/64` | `fd:1:200::ff` |
| Linux_6 | eth0      | `200` | `172.16.200.16/24` | `172.16.200.254` | `fd:1:200::16/64` | `fd:1:200::ff` |
| Linux_7 | eth0      | `200` | `172.16.200.17/24` | `172.16.200.254` | `fd:1:200::17/64` | `fd:1:200::ff` |
| Linux_8 | eth0      | `200` | `172.16.200.18/24` | `172.16.200.254` | `fd:1:200::18/64` | `fd:1:200::ff` |

---

<details>

<summary><h2>Настройка eBGP для Overlay сети в Underlay сети eBGP</h2></summary>

### Настройка eBGP на Spine

Настройка Underlay сети eBGP была произведена в [лаботаторной работе №4](../lab_04/README.md).

Произведем донастройку eBGP Overlay

<details>

<summary>Spine S01</summary>

```
peer-filter pf_LIAF
   10 match as-range 65001-65010 result accept
!
router bgp 65500
   router-id 192.168.1.1
   no bgp default ipv4-unicast
   timers bgp 3 9
   bgp listen range 192.168.0.0/24 peer-group pg_EVPN peer-filter pf_LIAF
   neighbor pg_EVPN peer group
   neighbor pg_EVPN next-hop-unchanged
   neighbor pg_EVPN update-source Loopback0
   neighbor pg_EVPN description OVERLAY
   neighbor pg_EVPN ebgp-multihop 4
   neighbor pg_EVPN send-community extended
   !
   address-family evpn
      neighbor pg_EVPN activate
!
```

Полная конфигурация [Spine S01](./conf/S01.eos)

</details>

<details>

<summary>Spine S02</summary>

```
peer-filter pf_LIAF
   10 match as-range 65001-65010 result accept
!
router bgp 65500
   router-id 192.168.1.2
   no bgp default ipv4-unicast
   timers bgp 3 9
   bgp listen range 192.168.0.0/24 peer-group pg_EVPN peer-filter pf_LIAF
   neighbor pg_EVPN peer group
   neighbor pg_EVPN next-hop-unchanged
   neighbor pg_EVPN update-source Loopback0
   neighbor pg_EVPN description OVERLAY
   neighbor pg_EVPN ebgp-multihop 4
   neighbor pg_EVPN send-community extended
   !
   address-family evpn
      neighbor pg_EVPN activate
!
```

Полная конфигурация [Spine S02](./conf/S02.eos)

</details>

---

### Настройка eBGP Overlay на Leaf

<details>

<summary>Leaf L01</summary>

```
!
vlan 100
   name Client_100
!
vlan 200
   name Client_200
!
vrf instance VRF1
!
interface Ethernet7
   description Client_200
   switchport access vlan 200
!
interface Ethernet8
   description Client_100
   switchport access vlan 100
!
interface Vlan100
   vrf VRF1
   ip address 172.16.100.1/24
   ipv6 enable
   ipv6 address fd:1:100::1/64
   ip virtual-router address 172.16.100.254
   ipv6 virtual-router address fd:1:100::ff
!
interface Vlan200
   vrf VRF1
   ip address 172.16.200.1/24
   ipv6 enable
   ipv6 address fd:1:200::1/64
   ip virtual-router address 172.16.200.254
   ipv6 virtual-router address fd:1:200::ff
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 100 vni 10100
   vxlan vlan 200 vni 10200
   vxlan vrf VRF1 vni 11100
!
ip virtual-router mac-address 02:00:00:00:00:00
!
ip routing vrf VRF1
!
ipv6 unicast-routing vrf VRF1
!
!
router bgp 65001
   router-id 192.168.0.1
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor pg_EVPN peer group
   neighbor pg_EVPN remote-as 65500
   neighbor pg_EVPN update-source Loopback0
   neighbor pg_EVPN description OVERLAY
   neighbor pg_EVPN ebgp-multihop 4
   neighbor pg_EVPN send-community extended
   neighbor 192.168.1.1 peer group pg_EVPN
   neighbor 192.168.1.2 peer group pg_EVPN
   !
   vlan 100
      rd auto
      route-target both 100:10100
      redistribute learned
   !
   vlan 200
      rd auto
      route-target both 200:10200
      redistribute learned
   !
   address-family evpn
      neighbor pg_EVPN activate
   !
   vrf VRF1
      rd 65001:1
      route-target import evpn 1:11100
      route-target export evpn 1:11100
      redistribute connected
!

```

Полная конфигурация [Leaf L01](./conf/L01.eos)

</details>

<details>

<summary>Leaf L02</summary>

```
!
vlan 100
   name Client_100
!
vlan 200
   name Client_200
!
vrf instance VRF1
!
interface Ethernet7
   description Client_200
   switchport access vlan 200
!
interface Ethernet8
   description Client_100
   switchport access vlan 100
!
interface Vlan100
   vrf VRF1
   ip address 172.16.100.2/24
   ipv6 enable
   ipv6 address fd:1:100::2/64
   ip virtual-router address 172.16.100.254
   ipv6 virtual-router address fd:1:100::ff
!
interface Vlan200
   vrf VRF1
   ip address 172.16.200.2/24
   ipv6 enable
   ipv6 address fd:1:200::2/64
   ip virtual-router address 172.16.200.254
   ipv6 virtual-router address fd:1:200::ff
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 100 vni 10100
   vxlan vlan 200 vni 10200
   vxlan vrf VRF1 vni 11100
!
ip virtual-router mac-address 02:00:00:00:00:00
!
ip routing vrf VRF1
!
ipv6 unicast-routing vrf VRF1
!
router bgp 65002
   router-id 192.168.0.2
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor pg_EVPN peer group
   neighbor pg_EVPN remote-as 65500
   neighbor pg_EVPN update-source Loopback0
   neighbor pg_EVPN description OVERLAY
   neighbor pg_EVPN ebgp-multihop 4
   neighbor pg_EVPN send-community extended
   neighbor 192.168.1.1 peer group pg_EVPN
   neighbor 192.168.1.2 peer group pg_EVPN
   !
   vlan 100
      rd auto
      route-target both 100:10100
      redistribute learned
   !
   vlan 200
      rd auto
      route-target both 200:10200
      redistribute learned
   !
   address-family evpn
      neighbor pg_EVPN activate
   !
   vrf VRF1
      rd 65002:1
      route-target import evpn 1:11100
      route-target export evpn 1:11100
      redistribute connected
!

```

Полная конфигурация [Leaf L02](./conf/L02.eos)

</details>

<details>

<summary>Leaf L03</summary>

```
!
vlan 100
   name Client_100
!
vlan 200
   name Client_200
!
vrf instance VRF1
!
interface Ethernet7
   description Client_200
   switchport access vlan 200
!
interface Ethernet8
   description Client_100
   switchport access vlan 100
!
interface Vlan100
   vrf VRF1
   ip address 172.16.100.3/24
   ipv6 enable
   ipv6 address fd:1:100::3/64
   ip virtual-router address 172.16.100.254
   ipv6 virtual-router address fd:1:100::ff
!
interface Vlan200
   vrf VRF1
   ip address 172.16.200.3/24
   ipv6 enable
   ipv6 address fd:1:200::3/64
   ip virtual-router address 172.16.200.254
   ipv6 virtual-router address fd:1:200::ff
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 100 vni 10100
   vxlan vlan 200 vni 10200
   vxlan vrf VRF1 vni 11100
!
ip virtual-router mac-address 02:00:00:00:00:00
!
ip routing vrf VRF1
!
ipv6 unicast-routing vrf VRF1
!
router bgp 65003
   router-id 192.168.0.3
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor pg_EVPN peer group
   neighbor pg_EVPN remote-as 65500
   neighbor pg_EVPN update-source Loopback0
   neighbor pg_EVPN description OVERLAY
   neighbor pg_EVPN ebgp-multihop 4
   neighbor pg_EVPN send-community extended
   neighbor 192.168.1.1 peer group pg_EVPN
   neighbor 192.168.1.2 peer group pg_EVPN
   !
   vlan 100
      rd auto
      route-target both 100:10100
      redistribute learned
   !
   vlan 200
      rd auto
      route-target both 200:10200
      redistribute learned
   !
   address-family evpn
      neighbor pg_EVPN activate
   !
   vrf VRF1
      rd 65003:1
      route-target import evpn 1:11100
      route-target export evpn 1:11100
      redistribute connected
!

```

Полная конфигурация [Leaf L03](./conf/L03.eos)

</details>

<details>

<summary>Leaf L04</summary>

```
!
vlan 100
   name Client_100
!
vlan 200
   name Client_200
!
vrf instance VRF1
!
interface Ethernet7
   description Client_200
   switchport access vlan 200
!
interface Ethernet8
   description Client_100
   switchport access vlan 100
!
interface Vlan100
   vrf VRF1
   ip address 172.16.100.4/24
   ipv6 enable
   ipv6 address fd:1:100::4/64
   ip virtual-router address 172.16.100.254
   ipv6 virtual-router address fd:1:100::ff
!
interface Vlan200
   vrf VRF1
   ip address 172.16.200.4/24
   ipv6 enable
   ipv6 address fd:1:200::4/64
   ip virtual-router address 172.16.200.254
   ipv6 virtual-router address fd:1:200::ff
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 100 vni 10100
   vxlan vlan 200 vni 10200
   vxlan vrf VRF1 vni 11100
!
ip virtual-router mac-address 02:00:00:00:00:00
!
ip routing vrf VRF1
!
ipv6 unicast-routing vrf VRF1
!
router bgp 65004
   router-id 192.168.0.4
   no bgp default ipv4-unicast
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor pg_EVPN peer group
   neighbor pg_EVPN remote-as 65500
   neighbor pg_EVPN update-source Loopback0
   neighbor pg_EVPN description OVERLAY
   neighbor pg_EVPN ebgp-multihop 4
   neighbor pg_EVPN send-community extended
   neighbor 192.168.1.1 peer group pg_EVPN
   neighbor 192.168.1.2 peer group pg_EVPN
   !
   vlan 100
      rd auto
      route-target both 100:10100
      redistribute learned
   !
   vlan 200
      rd auto
      route-target both 200:10200
      redistribute learned
   !
   address-family evpn
      neighbor pg_EVPN activate
   !
   vrf VRF1
      rd 65004:1
      route-target import evpn 1:11100
      route-target export evpn 1:11100
      redistribute connected
!
```

Полная конфигурация [Leaf L04](./conf/L04.eos)

</details>

</details>

## Проверка работоспособности eBGP Overlay сети и EVPN L3

<details>

<summary>Маршруты MAC/IP</summary>

### L01

!["Маршруты MAC/IP"](./img/L01_evpn_t2.png)

### L02

!["Маршруты MAC/IP"](./img/L02_evpn_t2.png)

### L03

!["Маршруты MAC/IP"](./img/L03_evpn_t2.png)

### L04

!["Маршруты MAC/IP"](./img/L04_evpn_t2.png)

</details>

<details>

<summary>Информация об IP-prefix</summary>

### L01

!["Маршруты MAC/IP"](./img/L01_evpn_t3_ipv4.png)

!["Маршруты MAC/IP"](./img/L01_evpn_t3_ipv6.png)

### L02

!["Маршруты MAC/IP"](./img/L02_evpn_t3_ipv4.png)

!["Маршруты MAC/IP"](./img/L02_evpn_t3_ipv6.png)

### L03

!["Маршруты MAC/IP"](./img/L03_evpn_t3_ipv4.png)

!["Маршруты MAC/IP"](./img/L03_evpn_t3_ipv6.png)

### L04

!["Маршруты MAC/IP"](./img/L04_evpn_t3_ipv4.png)

!["Маршруты MAC/IP"](./img/L04_evpn_t3_ipv6.png)

</details>

<details>

<summary>Маршрутная информация в VRF</summary>

### L01

!["Маршруты MAC/IP"](./img/L01_ip_ro_ipv4.png)

!["Маршруты MAC/IP"](./img/L01_ip_ro_ipv6.png)

### L02

!["Маршруты MAC/IP"](./img/L01_ip_ro_ipv4.png)

!["Маршруты MAC/IP"](./img/L01_ip_ro_ipv6.png)

### L03

!["Маршруты MAC/IP"](./img/L01_ip_ro_ipv4.png)

!["Маршруты MAC/IP"](./img/L01_ip_ro_ipv6.png)

### L04

!["Маршруты MAC/IP"](./img/L01_ip_ro_ipv4.png)

!["Маршруты MAC/IP"](./img/L01_ip_ro_ipv6.png)

</details>

---

## Проверка связности между клиентами

<details>

<summary>Пинги во VLAN 100 от Linux_1 к другим клиентам</summary>

!["Пинги от Linux_1"](./img/ping_linux_1_vlan100.png)

</details>

<details>

<summary>Пинги во VLAN 200 от Linux_1 к другим клиентам</summary>

!["Пинги от Linux_5"](./img/ping_linux_1_vlan200.png)

</details>

<details>

<summary>Распределение <b>echo request</b> и <b>echo reply</b> по разным линкам</summary>

!["Пинги от Linux_5"](./img/ping_L01_inc3.png)

</details>

---

## Обмен маршрутной информации NLRI

<details>

<summary>EVPN NLRI Route Type 2</summary>

!["EVPN NLRI Route Type 2"](./img/L01_nlri_t2.png)

</details>

<details>

<summary>EVPN NLRI Route Type 5</summary>

!["EVPN NLRI Route Type 5"](./img/L01_nlri_t5.png)

</details>
