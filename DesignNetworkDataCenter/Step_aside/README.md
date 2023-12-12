VRRP

C L2VPN без VRF работает

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback100
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp

R1
interface Ethernet0/0
 ip address 10.0.10.11 255.255.255.0
 vrrp 10 ip 10.0.10.254

R3
interface Ethernet0/0
 ip address 10.0.10.22 255.255.255.0
 vrrp 10 ip 10.0.10.254
 vrrp 10 priority 200

C L2VPN + VRF работает 



interface Vlan10
  no shutdown
  vrf member OTUS
  ip address 10.0.10.1/24
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member CLIENT
  ip address 10.0.11.1/24
  fabric forwarding mode anycast-gateway


interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback100
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
    
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 10020 l2
    rd auto
    route-target import auto
    route-target export auto

