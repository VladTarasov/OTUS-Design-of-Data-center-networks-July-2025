```
!
! Last configuration change at 13:31:05 UTC Wed Nov 5 2025
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
hostname COD2-FW
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
crypto pki trustpoint TP-self-signed-1982400059
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-1982400059
 revocation-check none
 rsakeypair TP-self-signed-1982400059
!
crypto pki trustpoint SLA-TrustPoint
 enrollment pkcs12
 revocation-check crl
!
!
crypto pki certificate chain TP-self-signed-1982400059
 certificate self-signed 01
  30820330 30820218 A0030201 02020101 300D0609 2A864886 F70D0101 05050030 
  31312F30 2D060355 04031326 494F532D 53656C66 2D536967 6E65642D 43657274 
  69666963 6174652D 31393832 34303030 3539301E 170D3235 31313035 31323336 
  34345A17 0D333531 31303531 32333634 345A3031 312F302D 06035504 03132649 
  4F532D53 656C662D 5369676E 65642D43 65727469 66696361 74652D31 39383234 
  30303035 39308201 22300D06 092A8648 86F70D01 01010500 0382010F 00308201 
  0A028201 0100A4CD 12325F09 10A11F42 987B8E16 3C740F72 AEEB1066 C9CD81A7 
  993BEA2D C5E24F60 55442A3D 532F1CF9 6A56473E DEF0C470 B7E6FAF2 3C817162 
  CF742D61 6683A3BF D40008A5 3645C61F 65BBECA3 8055E7B5 31DBFDA7 0305999E 
  D0254261 F3C79EFF DF19FEA0 B3E30FE3 35C2D1A7 D178B064 556A90C2 CD28C68D 
  B755F673 FF3A4692 2B19D996 031C291F C581AEF6 C7A3D7B7 D2E8FFEC 1C0591E6 
  E8A431C8 DB6EB115 D5F40439 47C21C08 0BF79D28 E13E9256 42E98772 845EF1F7 
  CCD01165 43543744 47FFA9F7 666FF7F9 8E0BF848 4EA7975B A70FD5FD 5090BC0B 
  AAEFC856 BA535046 FE229C0D DBB91DD7 1EB39DD2 59924DC5 A648252F 43B3BA41 
  F50724A6 E6E50203 010001A3 53305130 0F060355 1D130101 FF040530 030101FF 
  301F0603 551D2304 18301680 1459B2C3 E040F90A BA2E8148 65968EBC 64CD6AB8 
  63301D06 03551D0E 04160414 59B2C3E0 40F90ABA 2E814865 968EBC64 CD6AB863 
  300D0609 2A864886 F70D0101 05050003 82010100 77155992 9894E8C7 C5D142EF 
  0D80C0CD 2AF9613C E36C9C23 B09B02EE 86FD3FCE 5FA37E05 294A370F 18BADE15 
  4C9D3EE6 9DE16DBE 74E24C82 BB85DFD2 F7A636F9 811D3136 EA234535 7D2C5AA7 
  7C77E11B B2661761 9E5CB247 7A40310E AE3EEEF0 ECADFFAA F3040801 67F07641 
  D146CAC6 69C7F0F5 BDD6C465 2B05CA5A 838A2699 3FEDCA6E 88587D38 3A551C0A 
  BE93CAB5 699024E7 AE869070 87EB0856 FBB25FBD 1662A4C8 5F0CF997 BD855F3D 
  44E254EE 7B96249B 9C4117DB 1FB6A709 091F75FB BEEB636B 3CA4C602 21692408 
  33CA4438 2CB1938F A0005C1C 8DCF9954 9FFCEB4A 50A7E082 A18540BA 9BDBF677 
  66C08B6B 91D1F386 ED6DD134 1B9FF64A C36447FD
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
license udi pid ISRV sn 9F8HGKCJ8XP
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
 ip address 172.17.255.2 255.255.255.255
!
interface Port-channel1
 description =MH_Leaf1-2=
 no ip address
 no negotiation auto
 no mop enabled
 no mop sysid
!
interface Port-channel1.20
 description =WWW_GW=
 encapsulation dot1Q 20
 ip address 10.2.20.254 255.255.255.0
!
interface Port-channel1.100
 description =FW_Outside=
 encapsulation dot1Q 100
 ip address 172.17.250.1 255.255.255.254
!
interface Port-channel1.103
 description =Infrastructure=
 encapsulation dot1Q 103
 ip address 172.17.250.7 255.255.255.254
!
interface GigabitEthernet1
 description =Leaf1_Eth7=
 no ip address
 negotiation auto
 no mop enabled
 no mop sysid
 channel-group 1 mode active
!
interface GigabitEthernet2
 description =Leaf2_Eth7=
 no ip address
 negotiation auto
 no mop enabled
 no mop sysid
 channel-group 1 mode active
!
interface GigabitEthernet3
 no ip address
 shutdown
 negotiation auto
 no mop enabled
 no mop sysid
!
interface GigabitEthernet4
 no ip address
 shutdown
 negotiation auto
 no mop enabled
 no mop sysid
!
router bgp 65502
 bgp router-id 172.17.255.2
 bgp log-neighbor-changes
 redistribute connected route-map Services
 neighbor 172.17.250.0 remote-as 65502
 neighbor 172.17.250.0 timers 3 9
 neighbor 172.17.250.0 next-hop-self
 neighbor 172.17.250.0 soft-reconfiguration inbound
 neighbor 172.17.250.6 remote-as 65201
 neighbor 172.17.250.6 timers 3 9
 neighbor 172.17.250.6 soft-reconfiguration inbound
!
ip forward-protocol nd
ip http server
ip http authentication local
ip http secure-server
!
!
!
!
ip prefix-list Services seq 5 permit 10.2.20.0/24
!
!
route-map Services permit 10 
 match ip address prefix-list Services
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
