The machine has  two network interface cards. One is acting as an Wan and other is acting as an Lan.
Primary NIC:
eth0 - 192.168.168.27       #wan
Secondary NIC:
eth1 - 192.168.15.1         #lan
Another one network interface:
node1 - 192.168.15.3        #client
To Deploy Outboun  NAT Gateway on CentOS 7, follow the below steps:

Configure the kernel to forward IP packets:
nano /etc/sysctl.conf

# Controls IP packet forwarding
net.ipv4.ip_forward = 1

To avoid rebooting implement the same change dynamically:
sysctl -w net.ipv4.ip_forward=1

Clear Default Zone
firewall-cmd --set-default-zone=external
firewall-cmd --zone=internal --change-interface=eth1

On CentOS 7, after configuring both network interfaces, we need to use firewalld:
firewall-cmd --zone=external --add-interface=eth0 --permanent
firewall-cmd --zone=internal --add-interface=eth1 --permanent

After making changes reload with:
firewall-cmd --complete-reload

Check the settings to ensure your interfaces are listed in the correct zone:
firewall-cmd --list-all-zones

Configure masquerading on the externally facing device (eth0):
firewall-cmd --zone=external --add-masquerade --permanent

Now the NAT rule (see comments – this may not be required):
firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o eth0 -j MASQUERADE -s 192.168.15.0/24

I was running DNS, DHCP, pxe and several other services from my RTR001 machine to service the internal computers so I opened those ports with:
firewall-cmd --permanent --zone=internal --add-service=dhcp
firewall-cmd --permanent --zone=internal --add-service=tftp
firewall-cmd --permanent --zone=internal --add-service=dns
firewall-cmd --permanent --zone=internal --add-service=http
firewall-cmd --permanent --zone=internal --add-service=nfs
firewall-cmd --permanent --zone=internal --add-service=ssh

Reload the firewall rules and test pings from the internal machines:
firewall-cmd --complete-reload
firewall-cmd --list-all-zones

Enable NAT
firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o eth0 -j MASQUERADE -s 192.168.15.0/24
firewall-cmd --permanent --direct --passthrough ipv4 -I FORWARD -i eth1 -j ACCEPT
firewall-cmd --reload


Activate redirection
firewall-cmd --permanent --zone=internal --add-forward-port=port=80:proto=tcp:toport=3128:toaddr=192.168.15.1

firewall-cmd --permanent --zone=internal --add-port=3128/tcp

firewall-cmd --reload


Removing firewall from the machine

firewall-cmd --zone=external --remove-interface=eth0 --permanent
firewall-cmd --zone=internal --remove-interface=eth1 --permanent

firewall-cmd --zone=external --remove-masquerade --permanent

firewall-cmd --permanent --zone=internal --remove-service=dhcp
firewall-cmd --permanent --zone=internal --remove-service=tftp
firewall-cmd --permanent --zone=internal --remove-service=dns
firewall-cmd --permanent --zone=internal --remove-service=http
firewall-cmd --permanent --zone=internal --remove-service=nfs
firewall-cmd --permanent --zone=internal --remove-service=ssh

firewall-cmd --permanent --zone=internal --remove-forward-port=port=80:proto=tcp:toport=3128:toaddr=192.168.15.1

firewall-cmd --permanent --zone=internal --remove-port=3128/tcp



