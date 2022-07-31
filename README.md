# single-gpu-passthrough

These notes are specifically for my setup:
  * Arch Linux
  * GRUB2
  * Intel Core i9-12900KF
  * EVGA NVIDIA RTX 2080Ti (vBIOS: 90.02.30.00.98)
  * Samsung 870 EVO Plus
  * Intel X710 10Gb SFP+

  If your hardware or linux distribution differs, you will likely need to
  adjust some of these steps and/or configurations.

## Enabling IOMMU support in the kernel
Add the following two kernel command-line parameters by editing
`GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`
* `intel_iommu=on`
* `iommu=pt`

```shell
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 intel_iommu=on iommu=pt"
```

Generate a new grub configuration file with these changes.
```
# sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo grub-mkconfig -o /boot/grub/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/intel-ucode.img /boot/initramfs-linux.img
Adding boot menu entry for UEFI Firmware Settings ...
done
```

Reboot and confirm IOMMU is enabled and functioning.
```
# sudo dmesg | grep -i -e dmar -e iommu
[    0.007137] ACPI: DMAR 0x000000008E725D48 000070 (v01 INTEL  EDK2     00000002      01000013)
[    0.007161] ACPI: Reserving DMAR table memory at [mem 0x8e725d48-0x8e725db7]
[    0.044835] DMAR: IOMMU enabled
[    0.108073] DMAR: Host address width 39
[    0.108073] DMAR: DRHD base: 0x000000fed91000 flags: 0x1
[    0.108077] DMAR: dmar0: reg_base_addr fed91000 ver 1:0 cap d2008c40660462 ecap f050da
[    0.108079] DMAR: RMRR base: 0x0000008d398000 end: 0x0000008d3b7fff
[    0.108081] DMAR-IR: IOAPIC id 2 under DRHD base  0xfed91000 IOMMU 0
[    0.108082] DMAR-IR: HPET id 0 under DRHD base 0xfed91000
[    0.108083] DMAR-IR: Queued invalidation will be enabled to support x2apic and Intr-remapping.
[    0.109511] DMAR-IR: Enabled IRQ remapping in x2apic mode
[    0.292415] iommu: Default domain type: Passthrough (set via kernel command line)
[    0.394555] pci 0000:00:00.0: Adding to iommu group 0
[    0.394565] pci 0000:00:01.0: Adding to iommu group 1
...
[    0.394762] pci 0000:06:00.0: Adding to iommu group 17
[    0.394782] DMAR: Intel(R) Virtualization Technology for Directed I/O
```

In my configuration the GPU, SSD and audio device fall into the following
IOMMU groups:
```
IOMMU Group 13:
        00:1f.0 ISA bridge [0601]: Intel Corporation Device [8086:7a84] (rev 11)
        00:1f.3 Audio device [0403]: Intel Corporation Device [8086:7ad0] (rev 11)
        00:1f.4 SMBus [0c05]: Intel Corporation Device [8086:7aa3] (rev 11)
        00:1f.5 Serial bus controller [0c80]: Intel Corporation Device [8086:7aa4] (rev 11)
IOMMU Group 14:
        01:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti Rev. A] [10de:1e07] (rev a1)
        01:00.1 Audio device [0403]: NVIDIA Corporation TU102 High Definition Audio Controller [10de:10f7] (rev a1)
        01:00.2 USB controller [0c03]: NVIDIA Corporation TU102 USB 3.1 Host Controller [10de:1ad6] (rev a1)
        01:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU102 USB Type-C UCSI Controller [10de:1ad7] (rev a1)
IOMMU Group 16:
        03:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983 [144d:a808]
```

## Configure the VFIO Kernel Module
Create `/etc/modprobe.d/vfio.conf` and add the device identifiers to bind to.

> :warning: You must bind all of the deviecs within the IOMMU group,
even devices you do not want to passthrough to a guest.

* `10de:1e07` NVIDIA - VGA Controller
* `10de:10f7` NVIDIA - Audio Controller
* `10de:1ad6` NVIDIA - USB Controller
* `10de:1ad7` NVIDIA - USB Type-C Controller

* `8086:7a84` Intel - ISA bridge
* `8086:7ad0` Intel - Audio device
* `8086:7aa3` Intel - SMBus
* `8086:7aa4` Intel - Serial bus controller

* `144d:a808` Samsung - 870 EVO Plus

```
options vfio-pci ids=10de:1e07,10de:10f7,10de:1ad6,10de:1ad7,8086:7a84,8086:7ad0,8086:7aa3,8086:7aa4,144d:a808
options vfio-pci disable_vga=1
```
