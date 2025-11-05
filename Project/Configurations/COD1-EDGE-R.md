```
!
! Last configuration change at 14:38:11 UTC Wed Nov 5 2025
!
version 15.9
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname COD1-EDGE-R
!
boot-start-marker
boot-end-marker
!
!
vrf definition Infrastructure
 rd 172.16.255.1:101
 !
 address-family ipv4
 exit-address-family
!
no logging console
!
no aaa new-model
!
!
!
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
redundancy
!
!
! 
!
!
!
!
!
!
!
!
!
!
!
!
interface GigabitEthernet0/0
 no shutdown
 description =FW_Outside=
 ip address 172.16.250.3 255.255.255.254
 ip nat inside
 ip virtual-reassembly in
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 no shutdown
 description =Infrastructure=
 vrf forwarding Infrastructure
 ip address 172.16.250.5 255.255.255.254
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 no shutdown
 description =Internet=
 ip address dhcp
 ip nat outside
 ip virtual-reassembly in
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 no shutdown
 description =COD2-EDGE-R=
 vrf forwarding Infrastructure
 ip address 10.0.0.1 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
router bgp 65501
 bgp router-id 172.16.255.1
 bgp log-neighbor-changes
 redistribute static route-map S2B
 neighbor 172.16.250.2 remote-as 65501
 neighbor 172.16.250.2 description =FW_Outside=
 neighbor 172.16.250.2 timers 3 9
 neighbor 172.16.250.2 next-hop-self
 neighbor 172.16.250.2 soft-reconfiguration inbound
 default-information originate
 !
 address-family ipv4 vrf Infrastructure
  neighbor 10.0.0.2 remote-as 65502
  neighbor 10.0.0.2 description =COD2-EDGE-R=
  neighbor 10.0.0.2 timers 3 9
  neighbor 10.0.0.2 activate
  neighbor 10.0.0.2 soft-reconfiguration inbound
  neighbor 10.0.0.2 prefix-list COD2_Infra in
  neighbor 10.0.0.2 prefix-list COD1_Infra out
  neighbor 172.16.250.4 remote-as 65501
  neighbor 172.16.250.4 description =COD1-CORE-SW=
  neighbor 172.16.250.4 timers 3 9
  neighbor 172.16.250.4 activate
  neighbor 172.16.250.4 soft-reconfiguration inbound
 exit-address-family
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip nat inside source list Services interface GigabitEthernet0/2 overload
ip route 10.1.10.0 255.255.255.0 172.16.250.2
!
ip access-list standard Services
 permit 10.1.0.0 0.0.255.255
!
!
ip prefix-list COD1_Infra seq 5 permit 10.1.10.0/24
!
ip prefix-list COD2_Infra seq 5 permit 10.2.10.0/24
!
ip prefix-list S2B seq 5 permit 0.0.0.0/0
ipv6 ioam timestamp
!
route-map S2B permit 10
 match ip address prefix-list S2B
!
!
!
control-plane
!
banner exec ^CC
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
banner incoming ^CC
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
banner login ^CC
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
!
line con 0
 privilege level 15
line aux 0
line vty 0 4
 login
 transport input none
!
no scheduler allocate
!
end
```
