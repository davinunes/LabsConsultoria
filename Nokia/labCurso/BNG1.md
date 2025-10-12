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
/configure router bgp group IBGP type internal
/configure router bgp group IBGP cluster cluster-id 1.1.1.1
/configure router bgp group IBGP export policy export-ibgp

/configure router bgp neighbor 200.200.0.2 group "IBGP"
/configure router bgp neighbor 200.200.0.3 group "IBGP"
/configure router bgp neighbor 200.200.0.4 group "IBGP"
/configure router bgp neighbor 200.200.0.5 group "IBGP"


/configure policy-options prefix-list rota-default prefix 0.0.0.0/0 type exact
/configure policy-options prefix-list rotas-operadora-01 prefix 80.80.0.0/16 type longer

/configure policy-options policy-statement export-ibgp entry 5 from prefix-list "rota-default" protocol name bgp
/configure policy-options policy-statement export-ibgp entry 5 action action-type accept next-hop self

/configure policy-options policy-statement export-ibgp entry 10 from prefix-list "rotas-operadora-01" protocol name bgp
/configure policy-options policy-statement export-ibgp entry 10 action action-type accept next-hop self

/configure policy-options policy-statement export-ibgp entry 11 from protocol name bgp
/configure policy-options policy-statement export-ibgp entry 11 action action-type accept


show router bgp summary all
show router bgp neighbor "172.16.10.1" advertised-routes
show router bgp neighbor "172.16.10.1" received-routes
show router route-table protocol bgp
show router bgp routes
show router bgp routes 0.0.0.0/0 hunt
```


## Configura 6PE - Tunnel IPv6
```
/configure router interface "to-operadora-1-ipv6" port 1/1/4:101
/configure router interface "to-operadora-1-ipv6" ipv6 address 2000:1:c000::2 prefix-length 126
/configure router interface "system"
/configure router interface "system" ipv6 address 2001:1111::1 prefix-length 128

/configure router dhcp-server dhcpv4 LOCAL-DHCPV4-SERVER admin-state enable

```

## EBGP6 Operadora
```
/configure router bgp group EBGPv6 type external peer-as 65500 family ipv6  true
/configure router bgp group EBGPv6 local-as as-number 65001
/configure router bgp group EBGPv6 import policy import-bgp
/configure router bgp group EBGPv6 export policy export-ebgp-v6
/configure router bgp neighbor "2000:1:c000::1" group "EBGPv6"

/configure router static-routes route 2001:1111::/32 route-type unicast blackhole admin-state enable

/configure policy-options prefix-list "IPs-internos-v6"  prefix 2001:1111::/32 type exact 
/configure policy-options policy-statement "export-ebgp-v6" entry 10 from prefix-list "IPs-internos-v6"
/configure policy-options policy-statement "export-ebgp-v6" entry 10 action action-type accept
/configure policy-options policy-statement "export-ebgp-v6" default-action action-type reject

commit
admin save

```

# BRAS

## Configurar BNG

```

/configure router radius server "Radius" address 10.4.1.2 secret radius accept-coa true
/configure aaa radius server-policy AAA servers timeout 10
/configure aaa radius server-policy AAA servers retry-count 5 router-instance "Base"
/configure aaa radius server-policy AAA servers source-address 200.200.0.1
/configure aaa radius server-policy AAA servers server 1 server-name Radius
/configure aaa radius server-policy "AAA" acct-on-off


/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT"
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" description "POLITICA_ACCOUNTING_PARA_ASSINANTES"
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" radius-server-policy "AAA"
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" session-id-format number
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" queue-instance-accounting interim-update false


/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" session-accounting admin-state enable
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" session-accounting interim-update true host-update true
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" update-interval interval 720
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" include-radius-attribute acct-authentic true
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" include-radius-attribute acct-authentic true acct-delay-time true acct-triggered-reason true error-code true called-station-id true circuit-id true delegated-ipv6-prefix true
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" include-radius-attribute framed-interface-id true framed-ip-address true framed-ip-netmask true framed-ipv6-prefix true framed-ipv6-route true framed-route true ipv6-address true mac-address true nas-identifier true nat-port-range true remote-id true sla-profile true std-acct-attributes true sub-profile true subscriber-id true tunnel-server-attrs true user-name true
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" include-radius-attribute calling-station-id
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" include-radius-attribute nas-port-id
/configure subscriber-mgmt radius-accounting-policy "POLITICA_ACCT" include-radius-attribute nas-port-type

/configure subscriber-mgmt radius-authentication-policy "POLITICA_AUTH"
/configure subscriber-mgmt radius-authentication-policy "POLITICA_AUTH" description "POLITICA_AUTENTICACAO_PARA_ASSINANTES"
/configure subscriber-mgmt radius-authentication-policy "POLITICA_AUTH" pppoe-access-method pap-chap
/configure subscriber-mgmt radius-authentication-policy "POLITICA_AUTH" radius-server-policy "AAA" fallback action user-db "LUDB-Fallback-only"
/configure subscriber-mgmt radius-authentication-policy "POLITICA_AUTH" include-radius-attribute access-loop-options true called-station-id true circuit-id true dhcp-options true dhcp-vendor-class-id true mac-address true nas-identifier true pppoe-service-name true sap-session-index true tunnel-server-attrs true
/configure subscriber-mgmt radius-authentication-policy "POLITICA_AUTH" include-radius-attribute  acct-session-id type session
/configure subscriber-mgmt radius-authentication-policy "POLITICA_AUTH" include-radius-attribute calling-station-id
/configure subscriber-mgmt radius-authentication-policy "POLITICA_AUTH" include-radius-attribute nas-port-id
/configure subscriber-mgmt radius-authentication-policy "POLITICA_AUTH" include-radius-attribute nas-port-type


/configure subscriber-mgmt local-user-db "LUDB-Fallback-only" admin-state enable
/configure subscriber-mgmt local-user-db "LUDB-Fallback-only" ppp match-list mac host default admin-state enable
/configure subscriber-mgmt local-user-db "LUDB-Fallback-only" ppp match-list mac host default host-identification  service-name "local-users"
/configure subscriber-mgmt local-user-db "LUDB-Fallback-only" ppp match-list mac host default ipv4 address pool primary "POOL-BNG-PPPoE"
/configure subscriber-mgmt local-user-db "LUDB-Fallback-only" ppp match-list mac host default ipv6 delegated-prefix-pool "POOL-IPv6-DEFAULT" slaac-prefix-pool "POOL-IPv6-DEFAULT" force-ipv6cp true

show aaa radius-server-policy "AAA“


/configure subscriber-mgmt sub-profile "SUB-DEFAULT" radius-accounting policy "POLITICA_ACCT" session-optimized-stop true
/configure subscriber-mgmt sla-profile Default description "perfil padrao"
/configure subscriber-mgmt sla-profile Default egress qos sap-egress policy-name 20
/configure subscriber-mgmt sla-profile Default host-limits overall 3
/configure subscriber-mgmt sla-profile Default ingress ip-filter "NAT-Deterministico"
/configure subscriber-mgmt sla-profile Default ingress qos sap-ingress policy-name 10
/configure subscriber-mgmt sub-ident-policy "SUBSC_ID" sla-profile-map use-direct-map-as-default true
/configure subscriber-mgmt sub-ident-policy "SUBSC_ID" sub-profile-map use-direct-map-as-default true
/configure subscriber-mgmt ppp-policy "POLITICA_PPP" cookies false ppp-authentication pap ppp-initial-delay true ppp-mtu 1500 unique-sid per-sap keepalive interval 20
/configure subscriber-mgmt msap-policy "MSAP-DEFAULT" sub-sla-mgmt subscriber-limit 131071 sub-ident-policy "SUBSC_ID" defaults sla-profile "Default" sub-profile "SUB-DEFAULT" subscriber-id  auto-id
/configure subscriber-mgmt msap-policy "MSAP-DEFAULT" sub-sla-mgmt subscriber-limit 131071 sub-ident-policy "SUBSC_ID" single-sub-parameters profiled-traffic-only true
/configure subscriber-mgmt msap-policy "MSAP-DEFAULT" ies-vprn-only-sap-parameters anti-spoof next-hop-ip-and-mac-addr

/configure qos sap-ingress "10" description "UPLOAD" policer 1 rate pir max cir max
/configure qos sap-ingress "10" fc af policer 1
/configure qos sap-ingress "10" fc be policer 1
/configure qos sap-ingress "10" fc ef policer 1
/configure qos sap-ingress "10" fc h1 policer 1
/configure qos sap-ingress "10" fc h2 policer 1
/configure qos sap-ingress "10" fc l1 policer 1
/configure qos sap-ingress "10" fc l2 policer 1
/configure qos sap-ingress "10" fc nc policer 1
/configure qos sap-egress "20" description "DOWNLOAD"
/configure qos sap-egress "20" queue 1 rate pir max cir max
/configure qos sap-egress "20" fc be queue 1

/configure filter match-list ip-prefix-list "ip-system" prefix 20.1.1.0/31 
/configure filter match-list ip-prefix-list "ip-system" prefix 200.200.0.0/22 
/configure filter match-list ip-prefix-list "ip-system" prefix 200.200.0.1/32 
/configure filter match-list ip-prefix-list "ip-system" prefix 200.200.0.2/32

/configure filter ip-filter "NAT-Deterministico" default-action accept filter-id 100 entry 1 match dst-ip ip-prefix-list "ip-system"
/configure filter ip-filter "NAT-Deterministico" default-action accept filter-id 100 entry 1 action accept
/configure filter ip-filter "NAT-Deterministico" default-action accept filter-id 100 entry 10 match src-ip address 100.64.0.0/16
/configure filter ip-filter "NAT-Deterministico" default-action accept filter-id 100 entry 10 action nat

commit 

/configure router dhcp-server dhcpv4 "LOCAL-DHCPV4-SERVER" admin-state enable
/configure router dhcp-server dhcpv4 "LOCAL-DHCPV4-SERVER" pool-selection use-gi-address scope pool
/configure router dhcp-server dhcpv4 "LOCAL-DHCPV4-SERVER" pool-selection use-pool-from-client
/configure router dhcp-server dhcpv4 "LOCAL-DHCPV4-SERVER" pool "POOL-BNG-PPPoE" description "Pool para teste PPPoE" minimum-free percent 3
/configure router dhcp-server dhcpv4 "LOCAL-DHCPV4-SERVER" pool "POOL-BNG-PPPoE" options option dns-server ipv4-address 8.8.8.8
/configure router dhcp-server dhcpv4 "LOCAL-DHCPV4-SERVER" pool "POOL-BNG-PPPoE" subnet 100.64.0.0/24 options option default-router ipv4-address 100.64.0.1
/configure router dhcp-server dhcpv4 "LOCAL-DHCPV4-SERVER" pool "POOL-BNG-PPPoE" subnet 100.64.0.0/24 address-range 100.64.0.0 end 100.64.0.255
/configure router dhcp-server dhcpv4 "LOCAL-DHCPV4-SERVER" pool "POOL-BNG-PPPoE" subnet 100.64.0.0/24 exclude-addresses 100.64.0.1 end 100.64.0.1

/configure router interface "system" ipv4 local-dhcp-server LOCAL-DHCPV4-SERVER

commit


```


## VPLS
```

```

