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
/configure router interface "to-sw1" ipv4 primary address 10.0.0.0 prefix-length 31
/configure router interface "to-operadora-1" port 1/1/4:100 ipv4 primary address 172.16.10.2 prefix-length 30 
commit


```


# BNG2
```bash

```

# SW1

```
[gl:/configure]
A:admin@7250IXR-e_01# info
 port 1/1/1 {
 admin-state enable
 }
 port 1/1/2 {
 admin-state enable
 ethernet {
 mode access
 encap-type dot1q
 }
 }
 port 1/1/3 {
 admin-state enable
 }
 port 1/1/4 {
 admin-state enable

 /configure global
 port 1/1/4 admin-state enable
 
```


 

# SW1


# SW1
