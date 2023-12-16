# L2VPN

## AToM

- Any Transport over MPLS named by Cisco
- AToM is an open standard arch
- With AToM, customer doesn't have to do anything:
    - Customer do not need IP to connect to SP
    - Customer can use any transport
    - Customer can have L2 connection between site over MPLS
    - Meaning that I can have a VLAN 20 on site-A and site-B interface connecting to SP also running access port VLAN 20
 
### Transport for Layer 2 Frame
- 2 solutions available:
    - Transport customer traffice over MPLS
    - Transport customer IP traffic over MPLS - not L3VPN but L2TPv3
- L2TPv3:
    - L2 tunneling protocol over IP
    - Encap with L2TPv3 header and shipped across IP network
    - Arch based on pseudowire - carry L2 traffic from edge-to-edge across packet-switched backbone - either MPLS or IP

- L2TPv3 component:
    - Create PSN Tunnel between PE
    - Inside PSN Tunnel you will have one or more pseudowire
    - The pseudowire connect to attachement circuit (ACs)
    - pseudowire listen to frame and forward to AC, remote AC received and decap then forward to dest pseudowire
 
### AToM Architecture

- In case of MPLS, PSN Tunnel is the LSP
- To multiplex the pseudowire onto one PSN, PE use another LDP label
- The label called VC or PW
- Additional header 8 bytes

### AToM configs

- PE#1:
```
mpls ldp router-id Loopback0 force
mpls label protocol ldp
pseudowire-class X
encapsulation mpls
!
!Interface facing the customer
!
interface Serial0/1/0
no ip address
encapsulation hdlc
xconnect x.x.x.x 100 pw-class X
! x.x.x.x should be the remote-peer LDP router-id
```

- PE#2
```
mpls ldp router-id Loopback0 force
mpls label protocol ldp
pseudowire-class X
encapsulation mpls
!
!Interface facing the customer
!
interface Serial0/1/0
no ip address
encapsulation hdlc
xconnect x.x.x.x 100 pw-class X
! x.x.x.x should be the remote-peer LDP router-id
```

- Validate with `show mpls l2transport vc 100`

### AToM configs - Ethernet

- There many other L2 type supporter over AToM but skipped as it not relevant at the time of writing
- Ethernet:
- PE#1:
```
pseudowire-class X
encapsulation mpls
!
! Custoemr facing interface
!
interface FastEthernet9/0/0
no ip address
xconnect X.X.X.X 2000 pw-class X
```

- PE#2
```
pseudowire-class X
encapsulation mpls
!
! Custoemr facing interface
!
interface FastEthernet9/0/0
no ip address
xconnect X.X.X.X 2000 pw-class X
```

- Ethernet Trunk:
  - Customer facing interface running trunk interface - subinterface
  - The physical interface run xconnect similar to the previous config

- Ethernet Xconnect for one vlan:
  - Customer facing interface running trunk interface - subinterface
  - Run xconnect under the subinterface - if you have many subinterface, then you will have VCs for each interfaces

- PE#1
```
interface FastEthernet9/0/0.1
encapsulation dot1Q 100
xconnect X.X.X.X 2000 pw-class X
!
! Custoemr facing interface
!
interface FastEthernet9/0/0.2
encapsulation dot1Q 200
xconnect X.X.X.X 2001 pw-class X
```

- PE#2
```
interface FastEthernet9/0/0.1
encapsulation dot1Q 100
xconnect X.X.X.X 2000 pw-class X
!
! Custoemr facing interface
!
interface FastEthernet9/0/0.2
encapsulation dot1Q 200
xconnect X.X.X.X 2001 pw-class X
```

- QinQ
  - Customer have many vlan, then you want to transport over 1 vlan only
  - Can efficiently use to VLAN running on the PE, locally significant

- CE - Customer routers
```
!
! Interface facing the SP PE
!
interface FastEthernet0/1
no ip address
!
interface FastEthernet0/1.1
encapsulation dot1Q 1
ip address 10.1.2.2 255.255.255.0
!
interface FastEthernet0/1.2
encapsulation dot1Q 2
ip address 10.1.4.2 255.255.255.0
```
- PE
```
vlan dot1q tag native
!
! Customer facing interface
interface FastEthernet2/3
no ip address
switchport
switchport access vlan 800
switchport mode dot1qtunnel
spanning-tree bpdufilter enable
!
interface Vlan800
no ip address
mpls l2transport route X.X.X.X 800
```

## VPLS

- Service that emulate LAN
- Issue with AToM is you have to fully meshed it to support full LAN - BUM traffic

### Configs
- VFI needs to have a unique name on the PE router
- PE#1
```
mpls label protocol ldp
mpls ldp router-id Loopback0 force
l2 vfi cust-1 manual
vpn id 1
!
! Neighbor of PE#2
!
neighbor X.X.X.2 encapsulation mpls
!
! Neighbor of PE#3
!
neighbor X.X.X.3 encapsulation mpls
!
! Customer facing interface
!
interface FastEthernet4/2
no ip address
switchport
switchport access vlan 111
spanning-tree bpdufilter enable
!
interface Vlan111
no ip address
xconnect vfi cust-1
```
Validate `show vfi cust-1`, validate targeted hello `show mpls ldp neighbor`

- Trunk port VPLS
- PE#1
```
! Customer facing interface
!
interface FastEthernet4/2
no ip address
switchport
switchport trunk encapsulation dot1q
switchport trunk allowed vlan 111,222
switchport mode trunk
!
l2 vfi cust-1-111 manual
vpn id 1
neighbor X.X.X.X encapsulation mpls
neighbor X.X.X.X encapsulation mpls
!
l2 vfi cust-1-222 manual
vpn id 2
neighbor X.X.X.X encapsulation mpls
neighbor X.X.X.X encapsulation mpls
!
interface Vlan111
no ip address
xconnect vfi cust-1-111
!
interface Vlan222
no ip address
xconnect vfi cust-1-222
```
- QinQ
- PE#1
```
! Customer facing interface
!
interface FastEthernet4/2
no ip address
switchport
switchport access vlan 111
switchport mode dot1q-tunnel
spanning-tree bpdufilter enable
!
