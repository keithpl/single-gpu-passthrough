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
