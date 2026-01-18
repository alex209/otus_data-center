# Лабораторная №5

## VxLAN. EVPN L2 

### Цель задания

Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.

### Задачи

1. Настроите BGP peering между Leaf и Spine в AF l2vpn evpn
2. Настроите связанность между клиентами в первой зоне и убедитесь в её наличии
3. Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств

### Топология сети

![Топология сети](img/lab_04.png)


### Схема адресов IPv4 & IPv6

> **Примечание:** В текущей работе изменена адресация на линках Spine-Leaf с сетевой маски /30 на маску /31.

лан адресов IPv4 для линков Spline-Leaf составлен по схеме `10.a.b.c/31`, где

- a - номер DC/POD,
- b - номер Spine,
- c - по очереди для подсети /31.

Адреса loopback `192.168.x.y/32`, где

- x - 1 для Spine, 0 - для Leaf,
- y - номер spine или leaf по порядку

Адреса для клиентов - `172.16.x.y/24`, где

- x - номер VLAN,
- y - порядковый адрес хоста `+10`

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
| ------ | --------- | --------------   | ------------------    |
| L02    | Lo0       | `192.168.0.2/32` | `fdfd:1:1:0:2::/128`  |
|        | Eth1      | `10.1.1.3/31`    | `fdff:1:1:1:2::1/127` |
|        | Eth2      | `10.1.2.3/31`    | `fdff:1:1:2:2::1/127` |
| ------ | --------- | --------------   | ------------------    |
| L03    | Lo0       | `192.168.0.3/32` | `fdfd:1:1:0:3::/128`  |
|        | Eth1      | `10.1.1.5/31`    | `fdff:1:1:1:3::1/127` |
|        | Eth2      | `10.1.2.5/31`    | `fdff:1:1:2:3::1/127` |
| ------ | --------- | --------------   | ------------------    |
| L04    | Lo0       | `192.168.0.4/32` | `fdfd:1:1:0:4::/128`  |
|        | Eth1      | `10.1.1.7/31`    | `fdff:1:1:1:4::1/127` |
|        | Eth2      | `10.1.2.7/31`    | `fdff:1:1:2:4::1/127` |

#### Итоговая таблица адресов клиентов

| Device  | Interface | VLAN  | IP Address         | IPv6 Address      |
| ------- | --------- | ----  | ------------------ | ----------------- |
| Linux_1 | eth0      | `100` | `172.16.100.11/24` | `fd:1:100::11/64` |
| Linux_2 | eth0      | `100` | `172.16.100.12/24` | `fd:1:100::12/64` |
| Linux_3 | eth0      | `100` | `172.16.100.13/24` | `fd:1:100::13/64` |
| Linux_4 | eth0      | `100` | `172.16.100.14/24` | `fd:1:100::14/64` |
| Linux_5 | eth0      | `200` | `172.16.200.15/24` | `fd:1:200::15/64` |
| Linux_6 | eth0      | `200` | `172.16.200.16/24` | `fd:1:200::16/64` |
| Linux_7 | eth0      | `200` | `172.16.200.17/24` | `fd:1:200::17/64` |
| Linux_8 | eth0      | `200` | `172.16.200.18/24` | `fd:1:200::18/64` |


<details>

<summary><h2>Настройка iBGP для Overlay сети в Underlay сети iBGP</h2></summary>

### Настройка iBGP на Spine

Настройка Underlay сети iBGP была произведена в [лаботаторной работе №4](../lab_04/README.md).

Произведем донастройку IBGP 


<details>

<summary>S01</summary>

```
router bgp 65500
   router-id 192.168.1.1
   no bgp default ipv4-unicast
   bgp listen range 192.168.0.0/24 peer-group pg_EVPN remote-as 65500
   neighbor pg_EVPN peer group
   neighbor pg_EVPN update-source Loopback0
   neighbor pg_EVPN route-reflector-client
   neighbor pg_EVPN send-community extended
   address-family evpn
      neighbor pg_EVPN activate
```
Полная конфигурация [Spine S01](./conf/iBGP/S01.eos)

</details>

<details>

<summary>S02</summary>

```
router bgp 65500
   router-id 192.168.1.2
   no bgp default ipv4-unicast
   bgp listen range 192.168.0.0/24 peer-group pg_EVPN remote-as 65500
   neighbor pg_EVPN peer group
   neighbor pg_EVPN update-source Loopback0
   neighbor pg_EVPN route-reflector-client
   neighbor pg_EVPN send-community extended
   address-family evpn
      neighbor pg_EVPN activate
```
Полная конфигурация [Spine S02](./conf/iBGP/S02.eos)

</details>


</details>