# BNG1
## Configura Hardware

```bash
configure global
system name BNG1
commit
show card
card 1 card-type iom4-e
commit
show sfm
sfm 1 sfm-type m-sfm5-12e
sfm 2 sfm-type m-sfm5-12e
sfm 3 sfm-type m-sfm5-12e
commit
show card
card 1 mda 1 mda-type me10-10gb-sfp+
card 1 mda 2 mda-type isa2-bb
commit
admin save

/configure port 1/1/1 admin-state enable
/configure port 1/1/2 admin-state enable ethernet mode access encap-type qinq
/configure port 1/1/4 admin-state enable ethernet mode network encap-type dot1q
/configure port 1/1/7 admin-state enable ethernet mode access
/configure port 1/1/8 admin-state enable ethernet mode access
commit
show port
admin save

```
## Configura Interface
```bash
configure global
/configure router interface "system" ipv4 primary address 200.200.0.1 prefix-length 32
/configure router interface "to-sw1" port 1/1/1 ipv4 primary address 10.0.0.0 prefix-length 31
/configure router interface "to-operadora-1" port 1/1/4:100 ipv4 primary address 172.16.10.2 prefix-length 30 
commit
show router interface
```
## Configura OSPF
```bash
/configure router ospf 0 admin-state enable traffic-engineering true
/configure router ospf 0 area 0 interface "system"
/configure router ospf 0 area 0 interface "to-sw1" interface-type point-to-point
commit
show router ospf neighbor
```
## Configura MPLS
```bash
/configure router ldp admin-state enable
/configure router ldp interface-parameters interface "to-sw1" ipv4
commit
show router ldp session

```

## Configura RSVP
```bash
/configure routing-options if-attribute admin-group BLUE value 10
/configure routing-options if-attribute admin-group RED value 20
/configure router mpls admin-state enable
/configure router mpls interface "to-sw1"
/configure router rsvp admin-state enable
/configure router rsvp interface "to-sw1"
commit
admin save

```


## Configura LSPs
```bash
/configure router mpls path loose1 admin-state enable
/configure router mpls path loose2 admin-state enable

/configure router mpls lsp "to-BNG-02" admin-state enable
/configure router mpls lsp "to-BNG-02" type p2p-rsvp to 200.200.0.2
/configure router mpls lsp "to-BNG-02" path-computation-method local-cspf
/configure router mpls lsp "to-BNG-02" primary "loose1" exclude-admin-group group "RED"
/configure router mpls lsp "to-BNG-02" secondary "loose2" exclude-admin-group group "BLUE"

commit
admin save

show router mpls lsp "to-BNG-02" detail
show router mpls lsp "to-BNG-02" path detail
show router tunnel-table
oam lsp-trace rsvp-te lsp-name "to-BNG-02" path "loose1"
oam lsp-trace rsvp-te lsp-name "to-BNG-02" path "loose2"
show router rsvp session
show router rsvp session transit
show router ospf opaque-database
show router ospf opaque-database adv-router 200.200.0.5 detail
```

## Configura FRR
```
/configure router mpls lsp "to-BNG-02" fast-reroute frr-method one-to-one node-protect false
```

## Configura eBGP
```
/configure router autonomous-system 65501

/configure router bgp admin-state enable
/configure router bgp group EBGP peer-as 65500 type external
/configure router bgp group EBGP family ipv4 true
/configure router bgp group EBGP local-as as-number 65001
/configure router bgp group EBGP import policy import-bgp
/configure router bgp group EBGP export policy export-bgp

/configure router bgp neighbor 172.16.10.1 group "EBGP"

/configure router static-routes route 200.200.0.0/22 route-type unicast blackhole admin-state enable

/configure policy-options prefix-list IPs-internos prefix 200.200.0.0/22 type exact

/configure policy-options policy-statement export-bgp entry 10 from prefix-list "IPs-internos"
/configure policy-options policy-statement export-bgp entry 10 action action-type accept
/configure policy-options policy-statement export-bgp default-action  action-type reject

/configure policy-options policy-statement import-bgp entry 10 from protocol name bgp
/configure policy-options policy-statement import-bgp entry 10 action action-type accept
/configure policy-options policy-statement import-bgp default-action action-type reject

commit
admin save

```

## Configura iBGP
```

```



# BNG2
## Configura Hardware

```bash
configure global
system name BNG2
commit
show card
card 1 card-type iom4-e
commit
show sfm
sfm 1 sfm-type m-sfm5-12e
sfm 2 sfm-type m-sfm5-12e
sfm 3 sfm-type m-sfm5-12e
commit
show card
card 1 mda 1 mda-type me10-10gb-sfp+
card 1 mda 2 mda-type isa2-bb
commit
admin save

/configure port 1/1/1 admin-state enable
/configure port 1/1/2 admin-state enable ethernet mode access encap-type qinq
/configure port 1/1/4 admin-state enable ethernet mode network encap-type dot1q
/configure port 1/1/7 admin-state enable ethernet mode access
/configure port 1/1/8 admin-state enable ethernet mode access
commit
show port
admin save

```
## Configura Interface
```bash
configure global
/configure router interface "system" ipv4 primary address 200.200.0.2 prefix-length 32
/configure router interface "to-SW1" port 1/1/1 ipv4 primary address 10.0.0.5 prefix-length 31
/configure router interface "to-operadora-2" port 1/1/4:200 ipv4 primary address 172.16.20.2 prefix-length 30 
commit
show router interface
```
## Configura OSPF
```bash
/configure router ospf 0 admin-state enable traffic-engineering true
/configure router ospf 0 area 0 interface "system"
/configure router ospf 0 area 0 interface "to-SW1" interface-type  point-to-point
commit
show router ospf neighbor
```
## Configura MPLS
```bash
/configure router ldp admin-state enable
/configure router ldp interface-parameters interface "to-SW1" ipv4
commit
show router ldp session

```
## Configura RSVP
```bash
/configure routing-options if-attribute admin-group BLUE value 10
/configure routing-options if-attribute admin-group RED value 20
/configure router mpls admin-state enable
/configure router mpls interface "to-SW1"
/configure router rsvp admin-state enable
/configure router rsvp interface "to-SW1"
commit
admin save

show router mpls interface
```

## Configura LSPs
```bash
/configure router mpls path loose1 admin-state enable
/configure router mpls path loose2 admin-state enable

/configure router mpls lsp "to-BNG-01" admin-state enable
/configure router mpls lsp "to-BNG-01" type p2p-rsvp to 200.200.0.1
/configure router mpls lsp "to-BNG-01" path-computation-method local-cspf
/configure router mpls lsp "to-BNG-01" primary "loose1" exclude-admin-group group "RED"
/configure router mpls lsp "to-BNG-01" secondary "loose2" exclude-admin-group group "BLUE"

commit
admin save

show router mpls lsp "to-BNG-01" detail
show router mpls lsp "to-BNG-01" path detail
show router tunnel-table
```

## Configura FRR
```
/configure router mpls lsp "to-BNG-01" fast-reroute frr-method one-to-one node-protect false
```

## Configura eBGP
```
/configure router autonomous-system 65501

/configure router bgp admin-state enable
/configure router bgp group EBGP peer-as 65501 type external
/configure router bgp group EBGP family ipv4 true
/configure router bgp group EBGP local-as as-number 65001
/configure router bgp group EBGP import policy import-bgp
/configure router bgp group EBGP export policy export-bgp

/configure router bgp neighbor 172.16.20.1 group "EBGP"

/configure router static-routes route 200.200.0.0/22 route-type unicast blackhole admin-state enable

/configure policy-options prefix-list IPs-internos prefix 200.200.0.0/22 type exact

/configure policy-options policy-statement export-bgp entry 10 from prefix-list "IPs-internos"
/configure policy-options policy-statement export-bgp entry 10 action action-type accept
/configure policy-options policy-statement export-bgp default-action  action-type reject

/configure policy-options policy-statement import-bgp entry 10 from protocol name bgp
/configure policy-options policy-statement import-bgp entry 10 action action-type accept
/configure policy-options policy-statement import-bgp default-action action-type reject

commit
admin save

```

## Configura iBGP
```

```

# SW1
## Configura Hardware
```
configure global
system name Switch1
commit
/configure port 1/1/1 admin-state enable
/configure port 1/1/2 admin-state enable ethernet mode access encap-type dot1q
/configure port 1/1/3 admin-state enable
/configure port 1/1/4 admin-state enable
commit
admin save
show port
```
## Configura Interface
```bash
/configure router interface "system" ipv4 primary address 200.200.0.3 prefix-length 32
/configure router interface "to-BNG-1" port 1/1/1 ipv4 primary address 10.0.0.1 prefix-length 31
/configure router interface "to-SW-2" port 1/1/3 ipv4 primary address 10.0.0.2 prefix-length 31
/configure router interface "to-SW-3" port 1/1/4 ipv4 primary address 10.0.0.9 prefix-length 31
commit
admin save
```

## Configura OSPF
```bash
/configure router ospf 0 admin-state enable traffic-engineering true
/configure router ospf 0 area 0 interface "system"
/configure router ospf 0 area 0 interface "to-BNG-1" interface-type point-to-point
/configure router ospf 0 area 0 interface "to-SW-3" interface-type point-to-point
/configure router ospf 0 area 0 interface "to-sw-2" interface-type point-to-point
commit
admin save
 show router ospf neighbor
```

## Configura MPLS
```bash
/configure router ldp admin-state enable
/configure router ldp interface-parameters interface "to-BNG-1" ipv4
/configure router ldp interface-parameters interface "to-SW-3" ipv4
/configure router ldp interface-parameters interface "to-sw-2" ipv4
commit
show router ldp session
show router ldp bindings active
show router tunnel-table
```

## Configura RSVP
```bash
/configure routing-options if-attribute admin-group BLUE value 10
/configure routing-options if-attribute admin-group RED value 20
/configure router mpls admin-state enable
/configure router mpls interface "to-BNG-1"
/configure router mpls interface "to-sw-2" admin-group "BLUE"
/configure router mpls interface "to-SW-3" admin-group "RED"
/configure router rsvp admin-state enable
/configure router rsvp interface "to-BNG-1"
/configure router rsvp interface "to-sw-2"
/configure router rsvp interface "to-SW-3"
commit
admin save

show router mpls interface
```
 

# SW2
## Configura Hardware
```
configure global
system name Switch2
commit
/configure port 1/1/1 admin-state enable
/configure port 1/1/2 admin-state enable ethernet mode access encap-type dot1q
/configure port 1/1/3 admin-state enable
/configure port 1/1/4 admin-state enable
/configure port 1/1/5 admin-state enable ethernet mode access encap-type dot1q

commit
admin save
show port
```
## Configura Interface
```bash
/configure router interface "system" ipv4 primary address 200.200.0.4 prefix-length 32
/configure router interface "to-BNG-2" port 1/1/1 ipv4 primary address 10.0.0.4 prefix-length 31
/configure router interface "to-SW-1" port 1/1/3 ipv4 primary address 10.0.0.3 prefix-length 31
/configure router interface "to-SW-3" port 1/1/4 ipv4 primary address 10.0.0.6 prefix-length 31
commit
admin save
```

## Configura OSPF
```bash
/configure router ospf 0 admin-state enable traffic-engineering true
/configure router ospf 0 area 0 interface "system"
/configure router ospf 0 area 0 interface "to-BNG-2" interface-type point-to-point
/configure router ospf 0 area 0 interface "to-SW-3" interface-type point-to-point
/configure router ospf 0 area 0 interface "to-sw-1" interface-type point-to-point
commit
admin save
 show router ospf neighbor
```

## Configura MPLS
```bash
/configure router ldp admin-state enable
/configure router ldp interface-parameters interface "to-BNG-2" ipv4
/configure router ldp interface-parameters interface "to-SW-3" ipv4
/configure router ldp interface-parameters interface "to-sw-1" ipv4
commit
show router ldp session

```

## Configura RSVP
```bash
/configure routing-options if-attribute admin-group BLUE value 10
/configure routing-options if-attribute admin-group RED value 20
/configure router mpls admin-state enable
/configure router rsvp admin-state enable

/configure router mpls interface "to-BNG-2"
/configure router mpls interface "to-sw-1" admin-group "BLUE"
/configure router mpls interface "to-SW-3" admin-group "RED"

/configure router rsvp interface "to-BNG-2"
/configure router rsvp interface "to-sw-1"
/configure router rsvp interface "to-SW-3"

commit
admin save

show router mpls interface
```


# SW3
## Configura Hardware
```
configure global
system name Switch3
commit
/configure port 1/1/1 admin-state enable
/configure port 1/1/2 admin-state enable 
/configure port 1/1/3 admin-state enable ethernet mode access encap-type dot1q
commit
admin save
show port
```
## Configura Interface
```bash
/configure router interface "system" ipv4 primary address 200.200.0.5 prefix-length 32
/configure router interface "to-SW-2" port 1/1/1 ipv4 primary address 10.0.0.7 prefix-length 31
/configure router interface "to-SW-1" port 1/1/2 ipv4 primary address 10.0.0.8 prefix-length 31
commit
admin save
```

## Configura OSPF
```bash
/configure router ospf 0 admin-state enable traffic-engineering true
/configure router ospf 0 area 0 interface "system"
/configure router ospf 0 area 0 interface "to-SW-1" interface-type point-to-point
/configure router ospf 0 area 0 interface "to-SW-2" interface-type point-to-point
commit
admin save
 show router ospf neighbor
```

## Configura MPLS
```bash
/configure router ldp admin-state enable
/configure router ldp interface-parameters interface "to-SW-1" ipv4
/configure router ldp interface-parameters interface "to-SW-2" ipv4
commit
show router ldp session

```

## Configura RSVP
```bash
/configure routing-options if-attribute admin-group BLUE value 10
/configure routing-options if-attribute admin-group RED value 20
/configure router mpls admin-state enable
/configure router rsvp admin-state enable

/configure router mpls interface "to-SW-1" admin-group "RED"
/configure router mpls interface "to-SW-2" admin-group "RED"

/configure router rsvp interface "to-SW-1"
/configure router rsvp interface "to-SW-2"

commit
admin save

show router mpls interface
```
