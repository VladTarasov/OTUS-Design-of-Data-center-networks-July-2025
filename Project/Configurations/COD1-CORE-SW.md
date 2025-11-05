```
!
! Last configuration change at 14:31:45 UTC Wed Nov 5 2025
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname COD1-CORE-SW
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
!
!
!
!
!
no ip domain-lookup
ip cef
no ipv6 cef
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
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
interface Loopback0
 no shutdown
 ip address 172.16.255.3 255.255.255.255
!
interface GigabitEthernet0/0
 no shutdown
 description =DC=
 no switchport
 ip address 10.1.10.254 255.255.255.0
 negotiation auto
!
interface GigabitEthernet0/1
 no shutdown
 description =to_COD2=
 no switchport
 ip address 172.16.250.4 255.255.255.254
 negotiation auto
!
interface GigabitEthernet0/2
 no shutdown
 description =to_FW=
 no switchport
 ip address 172.16.250.1 255.255.255.254
 negotiation auto
!
interface GigabitEthernet0/3
 no shutdown
 negotiation auto
!
interface GigabitEthernet1/0
 no shutdown
 negotiation auto
!
interface GigabitEthernet1/1
 no shutdown
 negotiation auto
!
interface GigabitEthernet1/2
 no shutdown
 negotiation auto
!
interface GigabitEthernet1/3
 no shutdown
 negotiation auto
!
router bgp 65501
 bgp router-id 172.16.255.3
 bgp log-neighbor-changes
 redistribute connected route-map Infrastructure
 neighbor 172.16.250.5 remote-as 65501
 neighbor 172.16.250.5 timers 3 9
 neighbor 172.16.250.5 soft-reconfiguration inbound
!
ip forward-protocol nd
!
ip http server
!
ip route 0.0.0.0 0.0.0.0 172.16.250.0
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr
!
!
!
ip prefix-list Infrastructure seq 5 permit 10.1.10.0/24
!
!
route-map Infrastructure permit 10
 match ip address prefix-list Infrastructure
!
!
!
control-plane
!
banner exec ^C
IOSv - Cisco Systems Confidential -

Supplemental End User License Restrictions

This IOSv software is provided AS-IS without warranty of any kind. Under no circumstances may this software be used separate from the Cisco Modeling Labs Software that this software was provided with, or deployed or used as part of a production environment.

By using the software, you agree to abide by the terms and conditions of the Cisco End User License Agreement at http://www.cisco.com/go/eula. Unauthorized use or distribution of this software is expressly prohibited.
^C
banner incoming ^C
IOSv - Cisco Systems Confidential -

Supplemental End User License Restrictions

This IOSv software is provided AS-IS without warranty of any kind. Under no circumstances may this software be used separate from the Cisco Modeling Labs Software that this software was provided with, or deployed or used as part of a production environment.

By using the software, you agree to abide by the terms and conditions of the Cisco End User License Agreement at http://www.cisco.com/go/eula. Unauthorized use or distribution of this software is expressly prohibited.
^C
banner login ^C
IOSv - Cisco Systems Confidential -

Supplemental End User License Restrictions

This IOSv software is provided AS-IS without warranty of any kind. Under no circumstances may this software be used separate from the Cisco Modeling Labs Software that this software was provided with, or deployed or used as part of a production environment.

By using the software, you agree to abide by the terms and conditions of the Cisco End User License Agreement at http://www.cisco.com/go/eula. Unauthorized use or distribution of this software is expressly prohibited.
^C
!
line con 0
line aux 0
line vty 0 4
!
```
