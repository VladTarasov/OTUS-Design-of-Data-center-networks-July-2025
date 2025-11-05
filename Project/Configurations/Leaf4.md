```
! device: Leaf4 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model multi-agent
!
hostname Leaf4
!
spanning-tree mode mstp
!
vlan 10
   name =DC=
!
vlan 20
   name =WWW=
!
vrf instance Infrastructure
!
interface Port-Channel1
   description =WWW=
   switchport trunk allowed vlan 20
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:3401
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:34:01
   lacp system-id 0000.0000.3401
!
interface Port-Channel2
   description =COD2-DC=
   switchport trunk allowed vlan 10
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:3402
      designated-forwarder election algorithm preference 100
      route-target import 00:00:00:00:34:02
   lacp system-id 0000.0000.3402
!
interface Ethernet1
   description =Spine1_Eth4=
   mtu 9000
   no switchport
   ip address 172.17.201.7/31
!
interface Ethernet2
   description =Spine2_Eth4=
   mtu 9000
   no switchport
   ip address 172.17.202.7/31
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
   description =WWW_Gi0/1=
   channel-group 1 mode active
!
interface Ethernet8
   description =COD2-DC_Gi0/1=
   channel-group 2 mode active
!
interface Ethernet9
!
interface Loopback0
   description Underlay
   ip address 172.17.104.0/32
!
interface Management1
!
interface Vlan10
   vrf Infrastructure
   ip address virtual 10.2.10.254/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 20020
   vxlan vrf Infrastructure vni 777
!
ip routing
ip routing vrf Infrastructure
!
ip prefix-list Underlay seq 10 permit 172.17.104.0/32
!
route-map Underlay permit 10
   match ip address prefix-list Underlay
!
router bgp 65204
   router-id 172.17.104.0
   maximum-paths 16 ecmp 16
   neighbor Underlay peer group
   neighbor Underlay remote-as 65200
   neighbor Underlay timers 3 9
   neighbor Underlay password 7 pMz/0TkQX3Iq4ekhMDMo7Q==
   neighbor Underlay send-community extended
   neighbor 172.17.201.6 peer group Underlay
   neighbor 172.17.202.6 peer group Underlay
   redistribute connected route-map Underlay
   !
   vlan 10
      rd auto
      route-target import 65203:10010
      route-target export 65204:10010
   !
   vlan 20
      rd auto
      route-target import 65201:20020
      route-target import 65202:20020
      route-target import 65203:20020
      route-target export 65204:20020
      redistribute learned
   !
   address-family evpn
      neighbor Underlay activate
   !
   vrf Infrastructure
      rd 172.17.104.0:777
      route-target import evpn 65201:777
      route-target import evpn 65202:777
      route-target import evpn 65203:777
      route-target export evpn 65204:777
      redistribute connected
!
end
```
