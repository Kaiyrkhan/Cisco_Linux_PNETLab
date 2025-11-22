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

> SVI — L3 интерфейс, яғни VLAN-ның виртуалды routed интерфейсі

### Корпоративті желінің топологиясы
![Topology Enterprise Network Design](images/Topology_PNETLab_EnterpriseNetworkDesign_HQ1_v1_Cisco.png)
[Download Link for PNETLab Topology File](Topology/Topology_PNETLab_EnterpriseNetworkDesign_HQ1_v1_Cisco.zip)

### A1,A2 – Access Layer Switch-ті конфигурациялау
```shell
Switch> enable
Switch# configure terminal
Switch(config)# hostname A1
A1(config)#
```
