# Проектная работа

# Тема: "Проектирование мультиподовой сети ЦОД на базе технологии EVPN-VXLAN с безадресной IP-фабрикой (IPv6 Unnumbered BGP)"

## Цели:

 - Разработать масштабируемую архитектуру мультиподового ЦОД
 - Реализовать отказоустойчивый underlay с безадресной IPv6 маршрутизацией
 - Обеспечить мультитенантность и изоляцию клиентских сегментов
 - Реализовать мультихоминг (EVPN Ethernet Segment) для серверов
 - Организовать контролируемое межподовое взаимодействие
 - Подтвердить работоспособность и отказоустойчивость спроектированного решения

## Что планировалось (задачи проекта):

 - Разработать топологию Clos-фабрики с двумя подами.
 - Настроить Underlay: BGP unnumbered с использованием IPv6 link-local адресов на всех межкоммутаторных линках.
 - Реализовать Overlay: EVPN с VXLAN инкапсуляцией, распределённый anycast gateway, L2VNI и L3VNI для мультитенантности.
 - Настроить мультихоминг (EVPN Ethernet Segment) для подключения серверов к двум Leaf одновременно.
 - Организовать межподовое взаимодействие через Border Leaf с перезаписью route-target для единой маршрутизации между VRF разных подов.
 - Провести тестирование связности, отказоустойчивости и производительности.

## Используемые технологии

 - Underlay: IPv6 unnumbered, BGP (iBGP) с peer-group, ECMP.
 - Overlay: EVPN (RFC 7432), VXLAN (инкапсуляция), MP-BGP с address-family l2vpn evpn.
 - Мультитенантность: VRF (RED, BLUE), Route Distinguisher, Route Target.
 - Мультихоминг: EVPN Ethernet Segment (ESI), LACP.
 - Межподовое взаимодействие: Option A (VRF-to-VRF) на Border Leaf с перезаписью Route Target через route-map.
 - Платформа: Arista EOS (vEOS-lab).
  

## Архитектура сети (схема)

### Общая топология:

 - Два пода (Pod1: AS 65501, Pod2: AS 65502).
 - Каждый под: 2 Spine, 4 Leaf, 2 Border Leaf.
 - Связь между подами через линки Border Leaf (интерфейсы Ethernet8).
 - Серверы подключены к Leaf через Port-Channel (ESI-LAG).

### Underlay: 
 - Все линки между Leaf, Spine и Border Leaf – L3, с включённым IPv6 (link-local)
 - BGP-сессии устанавливаются через neighbor interface.

### Overlay: 
 
 - VXLAN туннели строятся от Leaf к Leaf, 
 - источник – Loopback0, 
 - Anycast gateway на SVI.