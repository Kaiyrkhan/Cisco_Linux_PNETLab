# Cisco-Based Enterprise Network Design and Implementation

### Тақырыбы: Cisco құрылғылары негізінде корпоративті желіні жобалау және конфигурациялау
### Жұмыстың орындалу қадамы: 
  1) VLAN;
  2) Link Aggregation. EtherChannel;
  3) Spanning Tree Protocol (STP);
  4) Switched Virtual Interface (SVI);
  5) HSRP;
  6) IP Address Configuration;
  7) Single area OSPFv2;
  8) ACL;
  9) NAT;
  10) Default Static Routing.

> SVI — L3 интерфейс, яғни VLAN-ның виртуалды routed интерфейсі (virtual routed interface)

### Корпоративті желінің топологиясы
![Topology Enterprise Network Design](images/Topology_PNETLab_EnterpriseNetworkDesign_HQ1_v1_Cisco.png)
[Download Link for PNETLab Topology File](Topology/Topology_PNETLab_EnterpriseNetworkDesign_HQ1_v1_Cisco.zip)

### A1, A2 – Access Layer Switch-ті конфигурациялау
```shell
Switch> enable
Switch# configure terminal

Switch(config)# hostname A1
A1(config)#

A1(config)# vlan 111
A1(config)# vlan 112
A1(config)# do show vlan brief

A1(config)# interface g1/1
A1(config)# switchport mode access
A1(config)# switchport access vlan 111

A1(config)# interface g1/2
A1(config)# switchport mode access
A1(config)# switchport access vlan 112

A1(config)# interface range g0/1-2
A1(config)# switchport mode trunk
A1(config)# switchport trunk allowed vlan 111,112
A1(config)# switchport nonegotiate

A1(config)# spanning-tree mode rapid-pvst
```

### D1, D2 – Distribution Layer Switch-ті конфигурациялау
```shell
Switch> enable
Switch# configure terminal

Switch(config)# hostname D1
D1(config)#
```

### C1 – Core Layer Switch-ті конфигурациялау
```shell
Switch> enable
Switch# configure terminal

Switch(config)# hostname C1
C1(config)#
```
