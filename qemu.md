QEMU

* https://linuxhint.com/install_qemu_debian/
* https://virtualchimp.wordpress.com/2016/03/09/creating-and-installing-windows-10-image-using-qemu/

Windows 10

```shell
mkdir -p ~/qemu/win10
qemu-img create -f qcow2 ~/qemu/win10/win10.qcow2 80G
qemu-system-x86_64 -cdrom ~/Win10_22H2_English_x64.iso -boot order=d -drive file=~/qemu/win10/win10.qcow2,format=qcow2 -m 8G -enable-kvm -machine q35 -device intel-iommu -cpu host -vnc :0 -nic user,hostfwd=tcp::3389-:3389
```