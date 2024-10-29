## Dual WAN configuration with recursive failover

**Reset Configuration**

    /system reset-configuration no-defaults=yes skip-backup=yes

**Create Lists**

    /interface list add name=WAN
    /interface list add name=LAN

**Rename interfaces** (optional)

    /interface ethernet set ether1 name=ether1-converge
    /interface ethernet set ether2 name=ether2-pldt
    /interface ethernet set ether6 name=ether6-desktop
    /interface ethernet set ether7 name=ether7-wifi
    /interface ethernet set ether8 name=ether8-homeserver
    /interface ethernet set ether9 name=ether9-smarttv

**Add the interfaces to their respective list**

    /interface list member add list=WAN interface=ether1-converge
    /interface list member add list=WAN interface=ether2-pldt
    /interface list member add list=LAN interface=ether6-desktop
    /interface list member add list=LAN interface=ether7-wifi
    /interface list member add list=LAN interface=ether8-homeserver
    /interface list member add list=LAN interface=ether9-smarttv

    :for i from=3 to=10 do={
      /interface list member add list=LAN interface=("ether" . $i)
    }

**Create LAN bridge**

    /interface bridge add name=bridge-lan
    /interface list member add list=LAN interface=bridge-lan

**Add the LAN interfaces to the bridge**

    /interface bridge port add bridge=bridge-lan interface=ether6-desktop
    /interface bridge port add bridge=bridge-lan interface=ether7-wifi
    /interface bridge port add bridge=bridge-lan interface=ether8-homeserver
    /interface bridge port add bridge=bridge-lan interface=ether9-smarttv

    :for i from=3 to=10 do={
      /interface bridge port add bridge=bridge-lan interface=("ether" . $i)
    }

**Configure IP for LAN**

    /ip address add address=192.168.88.1/24 interface=bridge-lan
    /ip pool add name=lan_pool ranges=192.168.88.2-192.168.88.254

**Configure DHCP server for LAN**

    /ip dhcp-server add address-pool=lan_pool interface=bridge-lan lease-time=10m name=lan_dhcp
    /ip dhcp-server network add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=8.8.8.8,8.8.4.4

**Configure DHCP for WAN**

    /ip dhcp-client add interface=ether1-converge use-peer-dns=no add-default-route=no
    /ip dhcp-client add interface=ether2-pldt use-peer-dns=no add-default-route=no

**Configure DNS**

    /ip dns set servers=8.8.8.8,8.8.4.4 allow-remote-requests=yes

**Configure NAT**

    /ip firewall nat add chain=srcnat out-interface-list=WAN action=masquerade 

**Configure recursive failover via IP Routes**

    /ip route add dst-address=8.8.8.8 gateway=192.168.1.1 check-gateway=ping distance=1 scope=10 target-scope=10 comment="pldt-monitor"
    /ip route add dst-address=0.0.0.0/0 gateway=8.8.8.8  check-gatway=ping distance=1   scope=10 target-scope=11 comment="pldt"

    /ip route add dst-address=4.2.2.1 gateway=192.168.100.1 distance=1 scope=10 target-scope=10 comment="converge-monitor"
    /ip route add dst-address=0.0.0.0/0 gateway=4.2.2.1 distance=2 scope=10 target-scope=11 comment="converge"




