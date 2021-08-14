# openwrt-kvm-libvirt
### Install notes for OpenWRT on QEMU/KVM, to manage and isolate the connectivity of my VMs to various VPNs and the internet in general.

OpenWRT is designed to be use on a router with a traditional WAN coming from a router, we will have to do some configuration to get into the web-based application. First you need to make sure you have just a basic serial connection in libvirt and a bridged network connection to your host machine. What I did was i just went ahead and added all the NICs i planned on using. The first is a bridged connection purely for configuration needs, a second as the default NAT in libvirt as my WAN connection and a third isolated network for my VMs to have internet connectivity and any extra NICs will be use to isolate VPN connections. In this repo are all the xml files and the openwrt .img file

Img file must be expanded before qemu will use it properly... 
```
git clone https://github.com/matsto641/openwrt-kvm-libvirt.git
cd openwrt-kvm-libvirt
gunzip openwrt-*.img.gz
qemu-img resize openwrt-*.img 300M
```
Now you can go ahead and import the xml files..
```
virsh define libvirtvm.xml
virsh net-define isolatedlan.xml
virsh net-define wan.xml
virsh net-define unmanaged.xml

```
And start the VM and enable NICs..
```
virsh net-start isolatedlan.xml
virsh net-start wan.xml
virsh net-start unmanaged.xml
virsh start libvirtvm.xml
```
Once your openwrt vm is up and running you'll need to gain access to the console, on the host machine enter the following...
```
sudo virsh console [openwrtVM] --safe
```
If you see this message congratulations you have successfully booted openwrt.
```
BusyBox v1.30.1 () built-in shell (ash)

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 -----------------------------------------------------
```
Now setup a static IP and subnet so you can configure your router.
```
ifconfig add 10.10.100.10
ifconfig netmask 255.255.255.0
```
Now you should be able to navigate to the ip you just defined in your browser on your host machine
http://10.10.100.10

TODO: SETUP NAT FORWARDING
TODO: SETUP OPENVPN RULES
TODO: SETUP AND CONFIGURE GUEST MACHINES

