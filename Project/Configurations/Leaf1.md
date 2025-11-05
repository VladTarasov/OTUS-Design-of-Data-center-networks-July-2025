```
! device: Leaf1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Leaf1
!
spanning-tree mode mstp
!
vlan 20
   name =WWW=
!
vlan 100
   name =FW_Outside=
!
vlan 101
   name =Infrastructure_Gate=
!
vlan 103
   name =Infrastructure_to_FW=
!
vrf instance Infrastructure
!
interface Port-Channel1
   description =COD2-FW=
   switchport trunk allowed vlan 20,100,103
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:1201
      designated-forwarder election algorithm preference 200
      route-target import 00:00:00:00:12:01
   lacp system-id 0000.0000.1201
!
interface Port-Channel2
   description =COD2-EDGE-R=
   switchport trunk allowed vlan 100-101
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:1202
      designated-forwarder election algorithm preference 200
      route-target import 00:00:00:00:12:02
   lacp system-id 0000.0000.1202
!
interface Ethernet1
   description =Spine1_Eth1=
   mtu 9000
   no switchport
   ip address 172.17.201.1/31
!
interface Ethernet2
   description =Spine2_Eth1=
   mtu 9000
   no switchport
   ip address 172.17.202.1/31
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
   description =COD2-FW_Gi0/0=
   channel-group 1 mode active
!
interface Ethernet8
   description =COD2-EDGE-R_Gi1=
   channel-group 2 mode active
!
interface Loopback0
   description Underlay
   ip address 172.17.101.0/32
!
interface Management1
!
interface Vlan101
   description =Infrastructure_Gate=
   vrf Infrastructure
   ip address 172.17.250.2/31
!
interface Vlan103
   description =to_FW=
   vrf Infrastructure
   ip address 172.17.250.6/31
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 20 vni 20020
   vxlan vlan 100 vni 700100
   vxlan vlan 101 vni 700101
   vxlan vlan 103 vni 700103
   vxlan vrf Infrastructure vni 777
!
ip routing
ip routing vrf Infrastructure
!
ip prefix-list Infrastructure
   seq 10 permit 10.2.10.0/24
!
ip prefix-list Underlay
   seq 10 permit 172.17.101.0/32
!
route-map Infrastructure permit 10
   match ip address prefix-list Infrastructure
!
route-map Underlay permit 10
   match ip address prefix-list Underlay
!
router bgp 65201
   router-id 172.17.101.0
   maximum-paths 16 ecmp 16
   neighbor Underlay peer group
   neighbor Underlay remote-as 65200
   neighbor Underlay timers 3 9
   neighbor Underlay password 7 pMz/0TkQX3Iq4ekhMDMo7Q==
   neighbor Underlay send-community extended
   neighbor 172.17.201.0 peer group Underlay
   neighbor 172.17.202.0 peer group Underlay
   redistribute connected route-map Underlay
   !
   vlan 100
      rd auto
      route-target import 65202:700100
      route-target export 65201:700100
      redistribute learned
   !
   vlan 101
      rd auto
      route-target import 65202:700101
      route-target export 65201:700101
      redistribute learned
   !
   vlan 103
      rd auto
      route-target import 65202:700103
      route-target export 65201:700103
      redistribute learned
   !
   vlan 20
      rd auto
      route-target import 65202:20020
      route-target import 65203:20020
      route-target import 65204:20020
      route-target export 65201:20020
      redistribute learned
   !
   address-family evpn
      neighbor Underlay activate
   !
   vrf Infrastructure
      rd 172.17.101.0:777
      route-target import evpn 65202:777
      route-target import evpn 65203:777
      route-target import evpn 65204:777
      route-target export evpn 65201:777
      neighbor 172.17.250.3 remote-as 65502
      neighbor 172.17.250.7 remote-as 65502
      redistribute connected route-map Infrastructure
!
end
```
