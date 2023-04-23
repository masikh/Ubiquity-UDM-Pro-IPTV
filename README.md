# Ubiquity-UDM-Pro-IPTV

Simple setup for enabling IPTV for CaiWay (Netherlands) on Ubiquity hardware

# Assumptions

Your physical WAN connection is port 11 (in software, this is eth10) using fiber-optics. And your 
tv setupbox is connected via port 2 which is assigned to vlan 101.

# Setup via GUI

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

# Setup via CLI (ssh)

In this section we'll assign vlan 101 to your WAN connection too. Unfortunately the Ubiquity GUI won't let you do this.
We need to assign vlan 101 to the WAN port too, because this is the network where the IPTV traffic will arrive.


#### Create an on boot script runner services

Ssh into your machine (your lan its gateway ip) and create a service unit (systemd) file 
/etc/systemd/system/udm-boot.service with the contents shown. This service unit will run the scripts present in 
the /mnt/data/on_boot.d folder at every boot.

    [Unit]
    Description=Run On Startup UDM
    Wants=network-online.target
    After=network-online.target
    
    [Service]
    Type=forking
    ExecStart=bash -c 'mkdir -p /mnt/data/on_boot.d && find -L /mnt/data/on_boot.d -mindepth 1 -maxdepth 1 -type f -print0 | sort -z | xargs -0 -r -n 1 -- bash -c \'if test -x "$0"; then echo "%n: running $0"; "$0"; else case "$0" in *.sh) echo "%n: sourcing $0"; . "$0";; *) echo "%n: ignoring $0";; esac; fi\''
    
    [Install]
    WantedBy=multi-user.target

Restart systemd so it will pickup your newly created script:

    systemctl daemon-reload

Enable the udm-boot service, so it will be run upon every reboot:

    systemctl enable udm-boot

Manually run udm-boot for test purposes (this will also create the /mnt/data/on_boot.d directory):

    systemctl start udm-boot

#### Add the on_boot script for tagging the WAN interface with vlan 101

Create a file /mnt/data/on_boot.d/enable_iptv.sh with the contents below. If your WAN interface is not port 11 (eth10)
then change it accordingly in the script below. Ubiquity counts the ports from zero, so 11 becomes 10 etc...

    #!/bin/bash
    WAN_INTERFACE=eth10
    IPTV_VLAN_ID=101
    IPTV_INTERFACE=$WAN_INTERFACE.$IPTV_VLAN_ID
    
    # ALIAS vconfig
    if ! command -v vconfig > /dev/null 2>&1; then
        alias udhcpc="busybox vconfig"
    fi
    
    # Setup vlan interface and bring it up
    vconfig add $WAN_INTERFACE $IPTV_VLAN_ID
    ifconfig $IPTV_INTERFACE up

After saving this file, make it executable

    chmod +x /mnt/data/on_boot.d/enable_iptv.sh

You can either run your enable_iptv.sh script manually or restart the udm-boot services

    systemctl restart udm-boot

# Check if everything is running as it should

After restarting your network-devices should be up and running.

    ifconfig | grep flags

Among other networks it should show

    br101: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
    eth10: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
    eth10.100: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
    eth10.101: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
    switch0.101: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500

where the eth10.100 is your tagged fiber WAN interface, eth10.101 your tagged fiber IPTV interface, br101 the 
bridge interface tying eth10.101 and switch0.101 together.

# Remove the power from your setupbox

Restart your setupbox by actually removing the power and plugging it back in. Your setupbox will boot and acquire an
IP-address from CaiWay. It might reboot a few times if new firmware has been found, but in the end you can watch tv.

Have fun.