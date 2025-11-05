```
!
! Last configuration change at 14:19:33 UTC Wed Nov 5 2025
!
version 15.9
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname COD1-FW
!
boot-start-marker
boot-end-marker
!
!
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
no ip icmp rate-limit unreachable
!
!
!
!
!
!
no ip domain lookup
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
no cdp log mismatch duplex
!
ip tcp synwait-time 5
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
interface Loopback0
 no shutdown
 ip address 172.16.255.2 255.255.255.255
!
interface GigabitEthernet0/0
 no shutdown
 description =Outside=
 ip address 172.16.250.2 255.255.255.254
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 no shutdown
 description =Infrastructure=
 ip address 172.16.250.0 255.255.255.254
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 no shutdown
 description =WWW=
 ip address 10.1.20.254 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 no shutdown
 no ip address
 shutdown
 duplex auto
 speed auto
 media-type rj45
!
router bgp 65501
 bgp router-id 172.16.255.2
 bgp log-neighbor-changes
 redistribute connected route-map Services
 neighbor 172.16.250.3 remote-as 65501
 neighbor 172.16.250.3 timers 3 9
 neighbor 172.16.250.3 soft-reconfiguration inbound
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 10.1.10.0 255.255.255.0 172.16.250.1
!
!
ip prefix-list Services seq 5 permit 10.1.20.0/24
ipv6 ioam timestamp
!
route-map Services permit 10
 match ip address prefix-list Services
!
!
!
control-plane
!
banner exec ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
banner incoming ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
banner login ^C
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
 exec-timeout 30 0
 privilege level 15
 logging synchronous
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line vty 0 4
 login
 transport input none
!
no scheduler allocate
!
end

```
