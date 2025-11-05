```
!
! Last configuration change at 13:47:06 UTC Wed Nov 5 2025
!
version 17.3
service timestamps debug datetime msec
service timestamps log datetime msec
! Call-home is enabled by Smart-Licensing.
service call-home
platform qfp utilization monitor load 80
platform punt-keepalive disable-kernel-core
platform console serial
!
hostname COD2-EDGE-R
!
boot-start-marker
boot-end-marker
!
!
vrf definition Infrastructure
 rd 172.17.255.1:101
 !
 address-family ipv4
 exit-address-family
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
no ip domain lookup
!
!
!
login on-success log
!
!
!
!
!
!
!
subscriber templating
! 
! 
! 
! 
!
!
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
crypto pki trustpoint TP-self-signed-2156571855
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-2156571855
 revocation-check none
 rsakeypair TP-self-signed-2156571855
!
crypto pki trustpoint SLA-TrustPoint
 enrollment pkcs12
 revocation-check crl
!
!
crypto pki certificate chain TP-self-signed-2156571855
 certificate self-signed 01
  30820330 30820218 A0030201 02020101 300D0609 2A864886 F70D0101 05050030 
  31312F30 2D060355 04031326 494F532D 53656C66 2D536967 6E65642D 43657274 
  69666963 6174652D 32313536 35373138 3535301E 170D3235 31313035 31323532 
  32395A17 0D333531 31303531 32353232 395A3031 312F302D 06035504 03132649 
  4F532D53 656C662D 5369676E 65642D43 65727469 66696361 74652D32 31353635 
  37313835 35308201 22300D06 092A8648 86F70D01 01010500 0382010F 00308201 
  0A028201 0100841B EFE18D4B 1748ABD2 8AE46671 0E121868 620C98DB 2CE04290 
  41CE32AB 72233413 4DA8A935 5FF3DD3E BB291CE8 631213E9 AFFC438E B1C670D6 
  E43643B7 8CA0AC90 942490C2 8A2B2DD0 C9FE4258 14434016 31AFE645 02E83BBA 
  876B856E 04B0B9A5 A1F55498 0A64CABC 3F18F7F4 C360D3FA BDCA785B E8BB7B15 
  EA571735 8F1A000F A7E66C22 744C3B14 BCB47F75 FDB38C53 C8BE9748 CC11E4D4 
  B7A1A33A BD5A9BE1 AF84183A 3EB4C4B7 B58F3D1E 50F1693E 21522249 A7F5A9AA 
  3E24FC46 F9D6E3E5 E8A7330D 06B92B92 A34D2C71 B0CB3C71 55D54DC6 A95D327B 
  7FE35595 BF9E7878 10C7C497 B3F46C78 EB4DEDFB D1EF59FF 84C2320E 7345E3DF 
  370854F5 742B0203 010001A3 53305130 0F060355 1D130101 FF040530 030101FF 
  301F0603 551D2304 18301680 14E57755 4F15D1E6 BB29AD16 AFF05A5A CCB2EBC2 
  9A301D06 03551D0E 04160414 E577554F 15D1E6BB 29AD16AF F05A5ACC B2EBC29A 
  300D0609 2A864886 F70D0101 05050003 82010100 0A523228 DFC8531E 3EB966E2 
  08D4D43B D0E2F6F3 E4443823 84F46184 CAF321A2 305C07DE 8D170B9D DBADA14A 
  8BE46BC4 E51ABB84 88BEE1C7 C141AE16 A7056371 FCDD652A AC081451 91DFEE18 
  B3801DA9 CC6D10C2 46E174AC B5966926 07F0002C D0269F65 1C18A744 BBD03CC0 
  3C7397FD CB9D9261 2BF9EBA2 E38E8FCA 6DB535C1 9CFF57F5 E0329016 222D8ADA 
  00A101E9 6CB8B3F0 637422FA 47E55496 1E32EC5F FD0A7830 2FC80B84 5D4BA0E9 
  99B65C01 FF29E461 BF979460 D4144378 5FEE8E3B AA167388 ED9A65E1 E08F6ED6 
  0C6A82C3 E17036C1 F7D42F3E 59A4E443 3DFC95AA 37B79E19 535107E8 8325A1D0 
  ED6ADA82 D8C9CC4A CF0BFB8D 5E975937 681E1BB9
  	quit
crypto pki certificate chain SLA-TrustPoint
 certificate ca 01
  30820321 30820209 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030 
  32310E30 0C060355 040A1305 43697363 6F312030 1E060355 04031317 43697363 
  6F204C69 63656E73 696E6720 526F6F74 20434130 1E170D31 33303533 30313934 
  3834375A 170D3338 30353330 31393438 34375A30 32310E30 0C060355 040A1305 
  43697363 6F312030 1E060355 04031317 43697363 6F204C69 63656E73 696E6720 
  526F6F74 20434130 82012230 0D06092A 864886F7 0D010101 05000382 010F0030 
  82010A02 82010100 A6BCBD96 131E05F7 145EA72C 2CD686E6 17222EA1 F1EFF64D 
  CBB4C798 212AA147 C655D8D7 9471380D 8711441E 1AAF071A 9CAE6388 8A38E520 
  1C394D78 462EF239 C659F715 B98C0A59 5BBB5CBD 0CFEBEA3 700A8BF7 D8F256EE 
  4AA4E80D DB6FD1C9 60B1FD18 FFC69C96 6FA68957 A2617DE7 104FDC5F EA2956AC 
  7390A3EB 2B5436AD C847A2C5 DAB553EB 69A9A535 58E9F3E3 C0BD23CF 58BD7188 
  68E69491 20F320E7 948E71D7 AE3BCC84 F10684C7 4BC8E00F 539BA42B 42C68BB7 
  C7479096 B4CB2D62 EA2F505D C7B062A4 6811D95B E8250FC4 5D5D5FB8 8F27D191 
  C55F0D76 61F9A4CD 3D992327 A8BB03BD 4E6D7069 7CBADF8B DF5F4368 95135E44 
  DFC7C6CF 04DD7FD1 02030100 01A34230 40300E06 03551D0F 0101FF04 04030201 
  06300F06 03551D13 0101FF04 05300301 01FF301D 0603551D 0E041604 1449DC85 
  4B3D31E5 1B3E6A17 606AF333 3D3B4C73 E8300D06 092A8648 86F70D01 010B0500 
  03820101 00507F24 D3932A66 86025D9F E838AE5C 6D4DF6B0 49631C78 240DA905 
  604EDCDE FF4FED2B 77FC460E CD636FDB DD44681E 3A5673AB 9093D3B1 6C9E3D8B 
  D98987BF E40CBD9E 1AECA0C2 2189BB5C 8FA85686 CD98B646 5575B146 8DFC66A8 
  467A3DF4 4D565700 6ADF0F0D CF835015 3C04FF7C 21E878AC 11BA9CD2 55A9232C 
  7CA7B7E6 C1AF74F6 152E99B7 B1FCF9BB E973DE7F 5BDDEB86 C71E3B49 1765308B 
  5FB0DA06 B92AFE7F 494E8A9E 07B85737 F3A58BE1 1A48A229 C37C1E69 39F08678 
  80DDCD16 D6BACECA EEBC7CF9 8428787B 35202CDC 60E4616A B623CDBD 230E3AFB 
  418616A9 4093E049 4D10AB75 27E86F73 932E35B5 8862FDAE 0275156F 719BB2F0 
  D697DF7F 28
  	quit
!
!
!
!
!
!
!
!
license udi pid ISRV sn 9XKS660NMKN
diagnostic bootup level minimal
memory free low-watermark processor 69838
!
!
spanning-tree extend system-id
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
 ip address 172.17.255.1 255.255.255.255
!
interface Port-channel1
 description =VRF_Leaks=
 no ip address
 no negotiation auto
 no mop enabled
 no mop sysid
!
interface Port-channel1.100
 description =FW_Outside=
 encapsulation dot1Q 100
 ip address 172.17.250.0 255.255.255.254
 ip nat inside
!
interface Port-channel1.101
 description =to_Infrastructure=
 encapsulation dot1Q 101
 vrf forwarding Infrastructure
 ip address 172.17.250.3 255.255.255.254
!
interface GigabitEthernet1
 description =Leaf1_Eth8=
 no ip address
 negotiation auto
 no mop enabled
 no mop sysid
 channel-group 1 mode active
!
interface GigabitEthernet2
 description =Leaf2_Eth8=
 no ip address
 negotiation auto
 no mop enabled
 no mop sysid
 channel-group 1 mode active
!
interface GigabitEthernet3
 description =COD1-EDGE-R=
 vrf forwarding Infrastructure
 ip address 10.0.0.2 255.255.255.252
 negotiation auto
 no mop enabled
 no mop sysid
!
interface GigabitEthernet4
 description =Internet=
 ip address dhcp
 ip nat outside
 negotiation auto
 no mop enabled
 no mop sysid
!
router bgp 65502
 bgp router-id 172.17.255.1
 bgp log-neighbor-changes
 redistribute static route-map S2B
 neighbor 172.17.250.1 remote-as 65502
 neighbor 172.17.250.1 description =FW_Outside=
 neighbor 172.17.250.1 timers 3 9
 neighbor 172.17.250.1 next-hop-self
 neighbor 172.17.250.1 soft-reconfiguration inbound
 default-information originate
 !
 address-family ipv4 vrf Infrastructure
  neighbor 10.0.0.1 remote-as 65501
  neighbor 10.0.0.1 description =COD1-EDGE-R=
  neighbor 10.0.0.1 timers 3 9
  neighbor 10.0.0.1 activate
  neighbor 10.0.0.1 soft-reconfiguration inbound
  neighbor 10.0.0.1 prefix-list COD1_Infra in
  neighbor 10.0.0.1 prefix-list COD2_Infra out
  neighbor 172.17.250.2 remote-as 65201
  neighbor 172.17.250.2 description =to_Infrastructure=
  neighbor 172.17.250.2 timers 3 9
  neighbor 172.17.250.2 activate
  neighbor 172.17.250.2 soft-reconfiguration inbound
 exit-address-family
!
ip forward-protocol nd
ip http server
ip http authentication local
ip http secure-server
!
ip nat inside source list Services interface GigabitEthernet4 overload
!
ip access-list standard Services
 10 permit 10.2.0.0 0.0.255.255
!
!
!
ip prefix-list COD1_Infra seq 5 permit 10.1.10.0/24
!
ip prefix-list COD2_Infra seq 5 permit 10.2.10.0/24
!
ip prefix-list S2B seq 5 permit 0.0.0.0/0
!
!
route-map S2B permit 10 
 match ip address prefix-list S2B
!
!
!
!
control-plane
!
!
mgcp behavior rsip-range tgcp-only
mgcp behavior comedia-role none
mgcp behavior comedia-check-media-src disable
mgcp behavior comedia-sdp-force disable
!
mgcp profile default
!
!
!
!
!
!
line con 0
 exec-timeout 30 0
 privilege level 15
 stopbits 1
line aux 0
 stopbits 1
line vty 0 4
 login
 transport input ssh
!
call-home
 ! If contact email address in call-home is configured as sch-smart-licensing@cisco.com
 ! the email address configured in Cisco Smart License Portal will be used as contact email address to send SCH notifications.
 contact-email-addr sch-smart-licensing@cisco.com
 profile "CiscoTAC-1"
  active
  destination transport-method http
!
!
!
!
!
end
```
