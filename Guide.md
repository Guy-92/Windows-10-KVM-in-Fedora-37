My system:

Distro: Fedora Linux 37 (Workstation Edition) x86_64 

Host: Lenovo Legion 5 15ARH05H

CPU: AMD Ryzen 7 4800H with Radeon Graphics (16) @ 2.900GHz  

GPU: NVIDIA GeForce GTX 1660 Ti Mobile

Muxed Configuration

Libvirt Version: 8.6.0

Please backup your XML before making these changes

# Adding NVIDIA Devices

1) First make sure your GPU is binded to VFIO, you can check by typing `lspci -ks 01:00.`

The output should list the "Kernel driver in use" as vfio-pci, for example, this is mine

```
01:00.0 VGA compatible controller: NVIDIA Corporation TU116M [GeForce GTX 1660 Ti Mobile] (rev a1)
	Subsystem: Lenovo Device 3a46
	Kernel driver in use: nvidia
	Kernel modules: nouveau, nvidia_drm, nvidia
01:00.1 Audio device: NVIDIA Corporation TU116 High Definition Audio Controller (rev a1)
	Subsystem: NVIDIA Corporation TU116 High Definition Audio Controller
	Kernel driver in use: snd_hda_intel
	Kernel modules: snd_hda_intel
01:00.2 USB controller: NVIDIA Corporation TU116 USB 3.1 Host Controller (rev a1)
	Subsystem: Lenovo Device 3a42
	Kernel driver in use: xhci_hcd
01:00.3 Serial bus controller: NVIDIA Corporation TU116 USB Type-C UCSI Controller (rev a1)
	Subsystem: Lenovo Device 3a42
	Kernel driver in use: nvidia-gpu
	Kernel modules: i2c_nvidia_gpu
```

2) Create a VM and add all the PCI devices with NVIDIA in it's name.

3) Copy the XML to a text editor, I used VS code. This makes it easier to find addresses using ctrl+f.

4) Replace the first line (domain type) in the XML with the line below
`<domain xmlns:qemu="http://libvirt.org/schemas/domain/qemu/1.0" type="kvm">`.

5) Remove the "address type" line for all the devices, except for the devices that isn't a part of the GPU. Meaning delete all lines that start with `        <address type`, that isn't a part of the GPU.

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

<address type="pci" omain="0x0000" bus="0x01" slot="0x00" function="0x2"/>

</hostdev>

<hostdev mode="subsystem" type="pci" managed="yes">

<source>

<address domain="0x0000" bus="0x01" slot="0x00" function="0x3"/>

</source>

<address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x3"/>

</hostdev>
```

Note: I'm not sure if the address type is the same in all laptops, the above step worked for me. Refer to the "How to get your address domain" section at the end to get your address domain.

6) Type this in your terminal `lspci -nks 01:00.0`

7) Convert the hex code of the two properties  "Subsystem" line to decimal using a converter like this one [Hexadecimal to Decimal Converter](https://www.rapidtables.com/convert/number/hex-to-decimal.html). So for example, in my laptop, the output is

```
01:00.0 0300: 10de:2191 (rev a1)
	Subsystem: 17aa:3a46
	Kernel driver in use: pci-stub
	Kernel modules: nouveau, nvidia_drm, nvidia
```

I converted the hex code "17aa", the result is "6058", and the hex code "3a46", which is "14918".

8) Add these lines to the end of the XML, in between `<devices/>` and `<domain/>`

```
  <qemu:override>
    <qemu:device alias="hostdev0">
      <qemu:frontend>
        <qemu:property name="x-pci-sub-vendor-id" type="unsigned" value="6058"/>
        <qemu:property name="x-pci-sub-device-id" type="unsigned" value="14918"/>
      </qemu:frontend>
    </qemu:device>
  </qemu:override>
```

11) Add the first converted hex code as the sub vendor id number, ie. replace "6058" with your first converted decimal number, and add the second converted hex code as the sub device id number, ie. replace "14918" with your second converted decimal number.


# Adding fake battery to your VM

1) Copy the following base64 string and paste it in this website. [base64.guru](https://base64.guru/converter/decode/file)

```
U1NEVKEAAAAB9EJPQ0hTAEJYUENTU0RUAQAAAElOVEwYEBkgoA8AFVwuX1NCX1BDSTAGABBMBi5f
U0JfUENJMFuCTwVCQVQwCF9ISUQMQdAMCghfVUlEABQJX1NUQQCkCh8UK19CSUYApBIjDQELcBcL
cBcBC9A5C1gCCywBCjwKPA0ADQANTElPTgANABQSX0JTVACkEgoEAAALcBcL0Dk=
```

2) Download the file, and rename it to "SSDT1.dat".

3) Move this file to your pool, for me I moved it to the "/var/lib/libvirt/images/pool" folder.

4) Add these lines in between `</devices>` and `<qemu:override>`

```
  <qemu:commandline>
    <qemu:arg value="-acpitable"/>
    <qemu:arg value="file=/var/lib/libvirt/images/pool/SSDT1.dat"/>
  </qemu:commandline>
```

5) Replace the path "/var/lib/libvirt/images/pool/SSDT1.dat" with the path to your SSDT1.dat file
6) Copy the full XML, go back to Virt-Manager, delete everything there and paste the edited XML, click apply and Virt-Manager will add the missing addresses.

  
# How to get your address domain
  
1) In the terminal, type `lspci -Dnn | grep VGA`

2) Make note of the PCI address of the NVIDIA GPU

3) In the terminal, type `virsh nodedev-dumpxml "pci_device"` and replace "pci_device" with `pci_$YOUR_PCI_DEVICE`, substituting the colons and decimals with underscores, for example, this is how I did it

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

4) Make sure the devices listed matches the address domains in your XML.

---

Guides I referenced,
[[GUIDE] GPU Passthrough for Laptop with Fedora · GitHub](https://gist.github.com/firelightning13/e530aec3e3a4e15885a10f6c4b7ae021#enable-iommu-grouping)

[NVIDIA GPU Passthrough on an Optimus MUXed Laptop | Lan Tian @ Blog](https://lantian.pub/en/article/modify-computer/laptop-muxed-nvidia-passthrough.lantian/?utm_source=pocket_saves)

[Beginner VFIO Tutorial - Part 0: Demo and Hardware](https://www.youtube.com/watch?v=fFz44XivxWI)

[20.23. Adding Multifunction PCI Devices to KVM Guest Virtual Machines Red Hat Enterprise Linux 7 | Red Hat Customer Portal](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-editing_a_guest_virtual_machines_configuration_file-adding_multifunction_pci_devices_to_kvm_guest_virtual_machines)

[PCI passthrough via OVMF - ArchWiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

---
