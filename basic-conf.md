# Configuração Mikrotik

## Interfaces
```bash
/interface bridge
add name=bridge1
/interface list
add comment=INTERNET name=WAN
add comment="REDE LOCAL" name=LAN
/interface bridge port
add bridge=bridge1 interface=ether3
add bridge=bridge1 interface=ether4
add bridge=bridge1 interface=ether5
/ip neighbor discovery-settings
set discover-interface-list=!dynamic
/interface list member
add interface=ether1 list=WAN
add interface=ether2 list=WAN
add interface=bridge1 list=LAN
```
## DHCP Server
```bash
/ip pool
add name=dhcp_pool0 ranges=192.168.0.100-192.168.0.199
/ip dhcp-server
add address-pool=dhcp_pool0 disabled=no interface=bridge1 name=dhcp1
/ip address
add address=192.168.0.1/24 interface=bridge1 network=192.168.0.0
/ip dhcp-server network
add address=192.168.0.0/24 dns-server=8.8.8.8,1.0.0.1 gateway=192.168.0.1
```
## DNS
```bash
/ip dns
set servers=208.67.222.222,208.67.220.220
```
## Clock/NTP Server
```bash
/system clock
set time-zone-name=America/Sao_Paulo
/system ntp client
set enabled=yes primary-ntp=200.160.7.186 secondary-ntp=201.49.148.135
```
## Firewall Address List
```bash
/ip firewall address-list
add address=192.168.0.0/24 comment=Traffic-LAN list=traffic_lan
add address=172.16.0.0/24 comment=Traffic-OpenVPN list=traffic_vpn
add address=0.0.0.0/8 comment="RFC6890-This Network" list=no_forward_ipv4
add address=10.0.0.0/8 comment=RFC1918-Private disabled=yes list=\
    no_forward_ipv4
add address=100.64.0.0/10 comment=RFC6598-CGNAT disabled=yes list=\
    no_forward_ipv4
add address=127.0.0.0/8 comment=RFC6890-LocalHost list=no_forward_ipv4
add address=169.254.0.0/16 comment=RFC6890-APIPA list=no_forward_ipv4
add address=172.16.0.0/12 comment=RFC1918-Private disabled=yes list=\
    no_forward_ipv4
add address=192.0.0.0/24 comment=RFC6890-IETF list=no_forward_ipv4
add address=192.0.2.0/24 comment=RFC6890-Documentation list=no_forward_ipv4
add address=192.168.0.0/16 comment=RFC1918-Private disabled=yes list=\
    no_forward_ipv4
add address=198.18.0.0/15 comment="RFC6890 Benchmark" list=no_forward_ipv4
add address=198.51.100.0/24 comment=RFC6890-Documentation list=\
    no_forward_ipv4
add address=203.0.113.0/24 comment=RFC6890-Documentation list=no_forward_ipv4
add address=224.0.0.0/4 comment=RFC5771-Multicast list=no_forward_ipv4
add address=240.0.0.0/4 comment=RFC6890-Reserved list=no_forward_ipv4
add address=255.255.255.255 comment=RFC6890-Broadcast list=no_forward_ipv4
add address=1.1.1.1 comment=DNS_Cloudflare-1 list=allow_dns
add address=1.0.0.1 comment=DNS_Cloudflare-2 list=allow_dns
add address=1.1.1.2 comment=DNS_Cloudflare_Block_Malware-1 list=allow_dns
add address=1.0.0.2 comment=DNS_Cloudflare_Block_Malware-2 list=allow_dns
add address=1.1.1.3 comment=DNS_Cloudflare_Block_Porn-1 list=allow_dns
add address=1.0.0.3 comment=DNS_Cloudflare_Block_Porn-2 list=allow_dns
add address=8.8.8.8 comment=DNS_Google-1 list=allow_dns
add address=8.8.4.4 comment=DNS_Google-2 list=allow_dns
add address=9.9.9.9 comment=DNS_Quad9-1 list=allow_dns
add address=149.112.112.112 comment=DNS_Quad9-2 list=allow_dns
add address=208.67.222.222 comment=Open_DNS-1 list=allow_dns
add address=208.67.220.220 comment=Open_DNS-2 list=allow_dns
add address=208.67.222.123 comment=Open_DNS_Block_Porn-1 list=allow_dns
add address=208.67.220.123 comment=Open_DNS_Block_Porn-2 list=allow_dns
```
## Firewall Filter
```bash
/ip firewall filter
add action=accept chain=input comment=\
    "Accept Established, Related, Untracked" connection-state=\
    established,related,untracked
add action=drop chain=input comment="Drop Invalid" connection-state=invalid
add action=accept chain=input comment="Accept ICMP Ping" protocol=icmp
add action=accept chain=input comment="Allow OpenVPN" dst-port=1194 \
    in-interface-list=WAN protocol=tcp
add action=accept chain=input dst-address=192.168.0.0/24 src-address=\
    172.16.0.0/24
add action=drop chain=input comment="Drop All Not Comming from LAN" \
    in-interface-list=!LAN
add action=accept chain=forward comment="Accept in ipsec policy" disabled=yes \
    ipsec-policy=in,ipsec
add action=accept chain=forward comment="Accept out ipsec policy" disabled=\
    yes ipsec-policy=out,ipsec
add action=accept chain=forward comment="Allow OpnVPN Traffic" dst-address=\
    192.168.0.0/24 src-address=172.16.0.0/24
add action=fasttrack-connection chain=forward comment="Fasttrack Connection"
add action=accept chain=forward comment=\
    "Accept Established, Related, Untracked" connection-state=\
    established,related,untracked
add action=accept chain=forward comment=Allow_Secure_DNS dst-address-list=\
    allow_dns dst-port=53 protocol=udp
add action=accept chain=forward comment=Allow_Secure_DNS dst-address-list=\
    allow_dns dst-port=53 protocol=tcp
add action=drop chain=forward comment=Deny_Others_DNS dst-port=53 log=yes \
    log-prefix=Deny-DNS protocol=udp src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_Others_DNS dst-port=53 protocol=\
    tcp src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_Telnet dst-port=23 protocol=tcp \
    src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_SMTP_Spammer dst-port=25 log=yes \
    log-prefix=Deny-Spammer protocol=tcp src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_TFTP dst-port=69 protocol=udp \
    src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_MS-RPC dst-port=135,593 protocol=\
    tcp src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_SMB dst-port=137-138 protocol=udp \
    src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_SMB dst-port=139,445 protocol=tcp \
    src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_SNMP dst-port=161-162 protocol=udp \
    src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_Syslog dst-port=514 protocol=udp \
    src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_SSDP dst-port=1900 protocol=udp \
    src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_LLMNR dst-port=5355 protocol=udp \
    src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_WinRM dst-port=5985-5986 protocol=\
    tcp src-address-list=traffic_lan
add action=drop chain=forward comment=Deny_IRC dst-port=6660-6669 protocol=\
    tcp src-address-list=traffic_lan
add action=drop chain=forward comment="Drop Invalid" connection-state=invalid
add action=drop chain=forward comment="Drop All from WAN not DSTNATed" \
    connection-nat-state=!dstnat connection-state=new in-interface-list=WAN
/ip firewall nat
add action=masquerade chain=srcnat out-interface-list=WAN
/ip firewall service-port
set ftp disabled=yes
set tftp disabled=yes
set irc disabled=yes
set udplite disabled=yes
```
## Dual WAN Netwatch
```bash
/ip address
add address=192.168.15.2/24 comment=WAN-1 interface=ether1 network=192.168.15.0
add address=192.168.1.2/24 comment=WAN-2 interface=ether2 network=192.168.1.0
```
```bash
/ip route
add comment=INET1 distance=1 gateway=192.168.15.1
add comment=INET2 distance=2 gateway=192.168.1.1
add comment=ROUTE-TEST distance=1 dst-address=1.1.1.1/32 gateway=192.168.15.1
```
```bash
/tool netwatch
add down-script="/ip route disable [find comment=INET1]" host=1.1.1.1 \
    interval=15s timeout=1s500ms up-script=\
    "/ip route enable [find comment=INET1]"
```
## Dual WAN Rota Recursiva
```bash
/ip address
add address=192.168.15.2/24 comment=WAN-1 interface=ether1 network=192.168.15.0
add address=192.168.1.2/24 comment=WAN-2 interface=ether2 network=192.168.1.0
```
```bash
/ip route
add check-gateway=ping distance=1 gateway=1.1.1.1
add comment=INET1 distance=1 dst-address=1.1.1.1/32 gateway=192.168.15.1 scope=10
add comment=INET2 distance=2 gateway=192.168.1.1
```
## OpenVPN

### CA
```bash
/certificate
add name=mkCA-template common-name=mkCA days-valid=3650 key-size=4096 \
key-usage=crl-sign,key-cert-sign
/certificate
sign mkCA-template name=mkCA
```
### Server Certificate
```bash
/certificate
add name=SERVER-template common-name=SERVER days-valid=3650 key-size=4096 \
key-usage=digital-signature,key-encipherment,data-encipherment,tls-server
/certificate
sign SERVER-template name=SERVER ca=mkCA
/certificate
set SERVIDOR trusted=yes
```
### Client Certificate
```bash
/certificate
add name=CLIENT-template common-name=CLIENT days-valid=365 key-size=4096 \
key-usage=tls-client
/certificate
sign CLIENT-template name=CLIENT ca=mkCA
```
### Export Certificates
```bash
/certificate
export-certificate mkCA export-passphrase=""
/certificate
export-certificate CLIENT export-passphrase=P@ssW0rdCert
```
### OpenVPN Server
```bash
/ip
pool add name="OpenVPN-Pool" ranges=172.16.0.2-172.16.0.254
```
```bash
/ppp
profile add name="OpenVPN-Profile" use-encryption=yes local-address=172.16.0.1 \
dns-server=9.9.9.9 remote-address=OpenVPN-Pool
```
```bash
/interface ovpn-server server
set enabled=yes port=1194 mode=ip netmask=24 default-profile=OpenVPN-Profile \
certificate=SERVER require-client-certificate=yes auth=sha1 cipher=aes256
```
### User OpenVPN
```bash
/ppp
secret add name=User-Name profile=OpenVPN-Profile password=P@ssW0rd service=ovpn
```
### DDNS 
```bash
/ip cloud
set ddns-enabled=yes ddns-update-interval=5m
```

## Regras MK firewall default
```bash
/ip firewall {
 filter add chain=input action=accept connection-state=established,related,untracked
comment="defconf: accept established,related,untracked"
 filter add chain=input action=drop connection-state=invalid comment="defconf: drop invalid"
 filter add chain=input action=accept protocol=icmp comment="defconf: accept ICMP"
 filter add chain=input action=accept dst-address=127.0.0.1 comment="defconf: accept to local
loopback (for CAPsMAN)"
 filter add chain=input action=drop in-interface-list=!LAN comment="defconf: drop all not
coming from LAN"
 filter add chain=forward action=accept ipsec-policy=in,ipsec comment="defconf: accept in ipsec
policy"
 filter add chain=forward action=accept ipsec-policy=out,ipsec comment="defconf: accept out
ipsec policy"
 filter add chain=forward action=fasttrack-connection connection-state=established,related
comment="defconf: fasttrack"
 filter add chain=forward action=accept connection-state=established,related,untracked
comment="defconf: accept established,related, untracked"
 filter add chain=forward action=drop connection-state=invalid comment="defconf: drop invalid"
 filter add chain=forward action=drop connection-state=new connection-nat-state=!dstnat ininterface-list=WAN comment="defconf: drop all from WAN not DSTNATed"
 }
```
### Opcional: Permitir acesso remoto WinBox para manutenção
```bash
/ip firewall filter add chain=input action=accept dst-port=8291 protocol=tcp in-interface-list=WAN \
    comment="custconf: accept WinBox"
```
