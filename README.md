### Multiple Windows VPS on Virtual dedicated server Using 5 IP address on hosting Providers platform over Debian Stretch with Proxmox Custom routed network, that enables RDP access to to each VM with seperate port.

###### Network Configuration done as following image. Configuration Prototype can be found In Official [Proxomox VE website], (https://pve.proxmox.com/wiki/Network_Configuration) 

![Prox111](https://user-images.githubusercontent.com/31945294/73002281-fbf7e880-3e2d-11ea-87b5-2f18745fd5bd.jpg)

```
# Config Dir /etc/network/interfaces

source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback

allow-hotplug eth0
iface eth1 inet manual

auto bond0
iface bond0 inet manual
      slaves eth0 eth1
      bond_miimon 100
      bond_mode 802.3ad
      bond_xmit_hash_policy layer2+3

iface bond0.3 inet manual
iface bond0.4 inet manual
iface bond0.5 inet manual
iface bond0.5 inet manual

post-up echo 1 > /proc/sys/net/ipv4/ip_forward
post-up echo 1 > /proc/sys/net/ipv4/conf/eth0/proxy_arp

auto vmbr0
iface vmbr0 inet static
	address	161.129.154.2
	netmask 255.255.255.248
	gateway 161.129.154.1
	broadcast 161.129.154.7
	bridge-ports bond0
	bridge-stp off
	bridge-fd 0
	dns-nameservers 1.0.0.1 64.6.64.6 209.244.0.3
	bridge_maxwait 0
	bridge_vlan_aware yes
	post-up ip route add 161.129.154.2/29 via 161.129.154.1 dev vmbr0
	post-up ip route add 161.129.154.3/29 via 161.129.154.1 dev vmbr0
	post-up ip route add 161.129.154.4/29 via 161.129.154.1 dev vmbr0
	
	# These 2 IP are set to VM/WIndows VPS 
	post-up ip route add 161.129.154.5/29 via 161.129.154.1 dev vmbr0
	post-up ip route add 161.129.154.6/29 via 161.129.154.1 dev vmbr0
	
auto vmbr0.3
iface vmbr0.3 inet static
	address	161.129.154.3
	netmask 29
	
auto vmbr0.4
iface vmbr0.4 inet static
	address	161.129.154.4
	netmask 29
	
auto vmbr3
iface vmbr3 inet static
	address 10.10.10.1
	netmask 255.255.255.0
	bridge-ports none
	bridge-stp off
	bridge-fd 0

auto vmbr4
iface vmbr4 inet static
	address 192.168.50.1
	netmask 255.255.255.0
	bridge-ports none
	bridge-stp off
	bridge-fd 0

#Allowing All Loopback traffic
post-up iptables -A INPUT -i lo -j ACCEPT
post-up iptables -A OUTPUT -o lo -j ACCEPT
	
#POSTROUTING with SNAT Starts Here
#Rules Snipet NAT :: vmbr3
post-up iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o vmbr0 -j SNAT --to-source 161.129.154.3
post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o vmbr0 -j SNAT --to-source 161.129.154.3
	
#Rules Snipet NAT :: vmbr4
post-up iptables -t nat -A POSTROUTING -s '192.168.50.0/24' -o vmbr0 -j SNAT --to-source 161.129.154.4
post-down iptables -t nat -A POSTROUTING -s '192.168.50.0/24' -o vmbr0 -j SNAT --to-source 161.129.154.4
### POSTROUTING with SNAT Ends Here
	
#PREROUTING with DNAT Starts here & is for RDP Access#
#vmbr3
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.3 -p tcp --dport 5001 -j DNAT --to 10.10.10.101:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.3 -p tcp --dport 5002 -j DNAT --to 10.10.10.102:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.3 -p tcp --dport 5003 -j DNAT --to 10.10.10.103:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.3 -p tcp --dport 5004 -j DNAT --to 10.10.10.104:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.3 -p tcp --dport 5005 -j DNAT --to 10.10.10.105:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.3 -p tcp --dport 5006 -j DNAT --to 10.10.10.106:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.3 -p tcp --dport 5007 -j DNAT --to 10.10.10.107:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.3 -p tcp --dport 5008 -j DNAT --to 10.10.10.108:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.3 -p tcp --dport 5009 -j DNAT --to 10.10.10.109:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.3 -p tcp --dport 5010 -j DNAT --to 10.10.10.110:3389
	
#vmbr4
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.4 -p tcp --dport 5021 -j DNAT --to 192.168.50.121:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.4 -p tcp --dport 5022 -j DNAT --to 192.168.50.122:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.4 -p tcp --dport 5023 -j DNAT --to 192.168.50.123:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.4 -p tcp --dport 5024 -j DNAT --to 192.168.50.124:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.4 -p tcp --dport 5025 -j DNAT --to 192.168.50.125:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.4 -p tcp --dport 5026 -j DNAT --to 192.168.50.126:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.4 -p tcp --dport 5027 -j DNAT --to 192.168.50.127:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.4 -p tcp --dport 5028 -j DNAT --to 192.168.50.128:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.4 -p tcp --dport 5029 -j DNAT --to 192.168.50.129:3389
post-up iptables -t nat -A PREROUTING -i vmbr0 -d 161.129.154.4 -p tcp --dport 5030 -j DNAT --to 192.168.50.130:3389
#PREROUTING Section Ends here & is for RDP Access#
	
#Everything Will MASQUERADE to vmbr0
	
post-up iptables -t nat -A POSTROUTING -o vmbr0 -j MASQUERADE
post-down iptables -t nat -D POSTROUTING -o vmbr0 -j MASQUERADE
	
#END
```

###### rules on >/etc/network/interfaces always loads at start up

###### With Above Configuration, Need to enable ip forwarding. Although the rules placed on above, for further confirmation u following code, required to be executed.
```
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding=1" >> /etc/sysctl.conf
sysctl -p
```
###### Finally Restart Networking
```
/etc/init.d/networking restart
```
