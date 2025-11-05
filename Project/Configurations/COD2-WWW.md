```
!
! Last configuration change at 15:35:28 UTC Mon Nov 3 2025
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname COD2-WWW
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
interface Port-channel1
 no shutdown
 description =MH_Leaf3-4=
 switchport trunk allowed vlan 20
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface GigabitEthernet0/0
 no shutdown
 description =Leaf3_Eth7=
 switchport trunk allowed vlan 20
 switchport trunk encapsulation dot1q
 switchport mode trunk
 negotiation auto
 channel-group 1 mode active
!
interface GigabitEthernet0/1
 no shutdown
 description =Leaf4_Eth7=
 switchport trunk allowed vlan 20
 switchport trunk encapsulation dot1q
 switchport mode trunk
 negotiation auto
 channel-group 1 mode active
!
interface GigabitEthernet0/2
 no shutdown
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
interface Vlan20
 no shutdown
 description =WWW_SRV=
 ip address 10.2.20.1 255.255.255.0
!
ip forward-protocol nd
!
ip http server
!
ip route 0.0.0.0 0.0.0.0 10.2.20.254
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr
!
!
!
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
 privilege level 15
line aux 0
line vty 0 4
 login
!

```
