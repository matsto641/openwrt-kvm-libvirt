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



Ive attached a copy of my libvirt XML settings..
```
<domain type="kvm">
  <name>alpinelinux3.13</name>
  <uuid>f16bac5b-ebba-419b-90fc-ed0ed419b64f</uuid>
  <metadata>
    <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
      <libosinfo:os id="http://alpinelinux.org/alpinelinux/3.13"/>
    </libosinfo:libosinfo>
  </metadata>
  <memory unit="KiB">786432</memory>
  <currentMemory unit="KiB">786432</currentMemory>
  <vcpu placement="static">2</vcpu>
  <os>
    <type arch="x86_64" machine="pc-i440fx-6.0">hvm</type>
  </os>
  <features>
    <acpi/>
    <apic/>
    <vmport state="off"/>
  </features>
  <cpu mode="host-passthrough" check="partial" migratable="on"/>
  <clock offset="utc">
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="pit" tickpolicy="delay"/>
    <timer name="hpet" present="no"/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled="no"/>
    <suspend-to-disk enabled="no"/>
  </pm>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <controller type="usb" index="0" model="qemu-xhci" ports="15">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x05" function="0x0"/>
    </controller>
    <controller type="pci" index="0" model="pci-root"/>
    <interface type="network">
      <mac address="52:54:00:0d:4f:9e"/>
      <source network="wrt"/>
      <model type="e1000"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x0"/>
    </interface>
    <interface type="network">
      <mac address="52:54:00:2a:8a:85"/>
      <source network="default"/>
      <model type="e1000"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x08" function="0x0"/>
    </interface>
    <interface type="network">
      <mac address="52:54:00:cd:3e:74"/>
      <source network="VPN-Internal"/>
      <model type="e1000"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x0b" function="0x0"/>
    </interface>
    <serial type="pty">
      <target type="isa-serial" port="0">
        <model name="isa-serial"/>
      </target>
    </serial>
    <console type="pty">
      <target type="serial" port="0"/>
    </console>
    <input type="mouse" bus="ps2"/>
    <input type="keyboard" bus="ps2"/>
    <audio id="1" type="spice"/>
    <hostdev mode="subsystem" type="usb" managed="yes">
      <source>
        <vendor id="0x13fe"/>
        <product id="0x6300"/>
      </source>
      <boot order="1"/>
      <address type="usb" bus="0" port="4"/>
    </hostdev>
    <memballoon model="virtio">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x09" function="0x0"/>
    </memballoon>
  </devices>
</domain>
```
