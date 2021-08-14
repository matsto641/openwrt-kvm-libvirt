# openwrt-kvm-libvirt
### Install notes for OpenWRT on QEMU/KVM, to manage and isolate the connectivity of my VMs to various VPNs and the internet in general.

OpenWRT .img file 
https://downloads.openwrt.org/releases/21.02.0-rc4/targets/x86/64/openwrt-21.02.0-rc4-x86-64-generic-ext4-combined.img.gz

Must be expanded before qemu will use it properly... 
```
gunzip openwrt-*.img.gz
qemu-img resize openwrt-*.img 300M
```
OpenWRT is designed to be use on a router with a traditional WAN coming from a router, we will have to do some configuration to get into the web-based application. First you need to make sure you have just a basic serial connection in libvirt and a bridged network connection to your host machine. What I did was i just went ahead and added all the NICs i planned on using. The first is a bridged connection purely for configuration needs, a second as the default NAT in libvirt as my WAN connection and a third isolated network for my VMs to have internet connectivity and any extra NICs will be use to isolate VPN connections.

Once your openwrt vm is up and running you'll need to gain access to the console, on the host machine enter the following...
```
sudo virsh console [openwrtVM] --safe
```



