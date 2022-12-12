My system:

Distro: Fedora Linux 37 (Workstation Edition) x86_64 

Host: Lenovo Legion 5 15ARH05H

CPU: AMD Ryzen 7 4800H with Radeon Graphics (16) @ 2.900GHz  

GPU: NVIDIA GeForce GTX 1660 Ti Mobile

Muxed Configuration

Libvirt Version: 8.6.0

---  

Please backup your XML before making these changes

# Adding NVIDIA Devices

1) First, make sure your GPU is binded to vfio-pci or pci-stub, you can check by typing `lspci -ks 01:00.`

The output should list the "Kernel driver in use" as vfio-pci or pci-stub, for example, this is mine

```
[user@legion ~]$ lspci -nks 01:00.
01:00.0 0300: 10de:2191 (rev a1)
	Subsystem: 17aa:3a46
	Kernel driver in use: pci-stub
	Kernel modules: nouveau, nvidia_drm, nvidia
01:00.1 0403: 10de:1aeb (rev a1)
	Subsystem: 10de:1aeb
	Kernel driver in use: pci-stub
	Kernel modules: snd_hda_intel
01:00.2 0c03: 10de:1aec (rev a1)
	Subsystem: 17aa:3a42
	Kernel driver in use: pci-stub
01:00.3 0c80: 10de:1aed (rev a1)
	Subsystem: 17aa:3a42
	Kernel driver in use: pci-stub
	Kernel modules: i2c_nvidia_gpu
```

2) Create a VM and add all the PCI devices with NVIDIA in it's name.

3) (optional) Copy the XML to a text editor, I used VS code. This makes it easier to find addresses using ctrl+f.

4) Replace the first line (domain type) in the XML with the line below
`<domain xmlns:qemu="http://libvirt.org/schemas/domain/qemu/1.0" type="kvm">`. This is so you can add QEMU arguments to the XML.

5) Remove the "address type" line for all the devices, except for the devices that isn't a part of the GPU. Meaning delete all lines that start with `        <address type`, that isn't a part of the GPU. This is so that no device address conflicts with the NVIDIA GPU devices addresses that you will set.

6) Replace the address type's "domain", "bus", "slot" and "function", with the source "domain", "bus", "slot" and "function", of all the NVIDIA GPU Devices.

For example, in my XML, I will change this
```
<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>

</source>

<address type="pci" domain="0x0000" bus="0x05" slot="0x00" function="0x0"/>

</hostdev>

<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x1"/>

</source>

<address type="pci" domain="0x0000" bus="0x06" slot="0x00" function="0x0"/>

</hostdev>

<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x2"/>

</source>

<address type="pci" domain="0x0000" bus="0x07" slot="0x00" function="0x0"/>

</hostdev>

<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x3"/>

</source>

<address type="pci" domain="0x0000" bus="0x08" slot="0x00" function="0x0"/>

</hostdev>
```

To this

```
<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>

</source>

<address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>

</hostdev>

<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x1"/>

</source>

<address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x1"/>

</hostdev>

<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x2"/>

</source>

<address type="pci" omain="0x0000" bus="0x01" slot="0x00" function="0x2"/>

</hostdev>

<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x3"/>

</source>

<address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x3"/>

</hostdev>
```

5) Add ` multifunction="on"` to the end of address type of the first GPU device, like this
```
<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>

</source>

<address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0" multifunction="on"/>

</hostdev>
```

My section after the changes looks like this

```
<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>

</source>

<address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0" multifunction="on"/>

</hostdev>

<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x1"/>

</source>

<address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x1"/>

</hostdev>

<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x2"/>

</source>

<address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x2"/>

</hostdev>

<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x3"/>

</source>

<address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x3"/>

</hostdev>
```


6) If you edited the XML in a text editor, copy the full XML, go back to Virt-Manager, delete everything there and paste the edited XML, click apply and Virt-Manager will add the missing addresses.

# How to get your address domain

The instructions below can be used to confirm your NVIDIA GPU address domain.

1) In the terminal, type `lspci -Dnn | grep VGA`

2) Make note of the PCI address of the NVIDIA GPU

3) In the terminal, type `virsh nodedev-dumpxml "pci_device"` and replace "pci_device" with the pci address of your NVIDIA GPU, substituting the colons and decimals with underscores, for example, this is how I did it

```
[user@legion ~]$ lspci -Dnn | grep VGA
0000:01:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU116M [GeForce GTX 1660 Ti Mobile] [10de:2191] (rev a1)
0000:05:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Renoir [1002:1636] (rev c6)
[user@legion ~]$ virsh nodedev-dumpxml pci_0000_01_00_0
<device>
  <name>pci_0000_01_00_0</name>
  <path>/sys/devices/pci0000:00/0000:00:01.1/0000:01:00.0</path>
  <parent>pci_0000_00_01_1</parent>
  <driver>
    <name>nvidia</name>
  </driver>
  <capability type='pci'>
    <class>0x030000</class>
    <domain>0</domain>
    <bus>1</bus>
    <slot>0</slot>
    <function>0</function>
    <product id='0x2191'>TU116M [GeForce GTX 1660 Ti Mobile]</product>
    <vendor id='0x10de'>NVIDIA Corporation</vendor>
    <iommuGroup number='9'>
      <address domain='0x0000' bus='0x01' slot='0x00' function='0x2'/>
      <address domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
      <address domain='0x0000' bus='0x01' slot='0x00' function='0x3'/>
      <address domain='0x0000' bus='0x01' slot='0x00' function='0x1'/>
    </iommuGroup>
  </capability>
</device>
```

4) Make sure the address domains listed matches the address domains in your XML.


Other things to do is adding the device and vendor ids, and adding a fake battery if needed. How to do them is mentioned in [firelightning13](https://gist.github.com/firelightning13)'s guide [here](https://gist.github.com/firelightning13/e530aec3e3a4e15885a10f6c4b7ae021).



---

Other guides/references

[[GUIDE] GPU Passthrough for Laptop with Fedora.md](https://gist.github.com/firelightning13/e530aec3e3a4e15885a10f6c4b7ae021)

[NVIDIA GPU Passthrough on an Optimus MUXed Laptop | Lan Tian @ Blog](https://lantian.pub/en/article/modify-computer/laptop-muxed-nvidia-passthrough.lantian/?utm_source=pocket_saves)

[Beginner VFIO Tutorial](https://www.youtube.com/playlist?list=PLG7vUqRxMOG6gsPXohhFht3UJbcCxYgcL)

[20.23. Adding Multifunction PCI Devices to KVM Guest Virtual Machines Red Hat Enterprise Linux 7 | Red Hat Customer Portal](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-editing_a_guest_virtual_machines_configuration_file-adding_multifunction_pci_devices_to_kvm_guest_virtual_machines)

[PCI passthrough via OVMF - ArchWiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

---
