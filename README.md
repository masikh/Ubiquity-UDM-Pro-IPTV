# Ubiquity-UDM-Pro-IPTV

Simple setup for enabling IPTV for CaiWay (Netherlands) on Ubiquity hardware

# Assumptions

Your physical WAN connection is port 11 (in software, this is eth10) using fiber-optics. And your 
tv setupbox is connected via port 2 which is assigned to vlan 101.

# Setup

CaiWay uses 3 vlans for their internet access.

    VLAN 100 Internet   
    VLAN 101 TV   
    VLAN 102 Voice 

Make sure your WAN connection is using a third party gateway and has vlan tag 100 assigned.
After a short while you'll receive a DHCP address on your WAN connection. 

![Primary WAN](/primary-wan.png "Primary WAN")

Secondly make a new network (I call it **VLAN101**) also with a third-party gateway.

![IPTV Network](/IPTV-Network.png "IPTV Network (VLAN101)")

Lastly assign network port 2 (or any other to you likening) which is connected to your tv setupbox.

![Port Profile](/Port-profile.png "Port Profile")

