# kubevirt-cpu-emulation
Testing the feasibility of running aarch64 guest on x86_64 host

```
dnf install -y guestfs-tools
```


```bash
fdisk -l 2023-12-05-raspios-bookworm-arm64-lite.img

echo $(( 8192*512 ))

mount -o loop,offset=4194304 2023-12-05-raspios-bookworm-arm64-lite.img /mnt
cp /mnt/kernel8.img ~/
cp /mnt/bcm2710-rpi-3-b-plus.dtb ~/
umount /mnt

qemu-img resize 2023-12-05-raspios-bookworm-arm64-lite.img 4G
```

```bash
qemu-system-aarch64 -machine raspi3b -cpu cortex-a72 -smp 4 -m 1G -kernel kernel8.img -append "root=/dev/mmcblk0p2 rootdelay=1 rootfstype=ext4 rw panic=0 console=ttyAMA0,115200" -sd 2023-12-05-raspios-bookworm-arm64-lite.img -dtb bcm2710-rpi-3-b-plus.dtb -display none -monitor telnet:127.0.0.1:5555,server,nowait -nographic -serial stdio

qemu-system-aarch64 -machine raspi3b -smp 4 -m 1G -kernel kernel8.img -append "rw loglevel=8 root=/dev/mmcblk0p2 rootdelay=1 rootfstype=ext4 rw panic=0 console=ttyAMA0,115200" -sd 2023-12-05-raspios-bookworm-arm64-lite.img -dtb bcm2710-rpi-3-b-plus.dtb -display none -monitor telnet:127.0.0.1:5555,server,nowait -nographic -serial stdio

qemu-system-aarch64 -M raspi3 -append "rw earlyprintk loglevel=8 console=ttyAMA0,115200 dwc_otg.lpm_enable=0 root=/dev/mmcblk0p2 rootdelay=1" -dtb bcm2710-rpi-3-b-plus.dtb -sd 2021-10-30-raspios-bullseye-armhf.img -kernel kernel8.img -m 1G -smp 4 -serial stdio -usb -device usb-mouse -device usb-kbd -vnc 192.168.2.1:0
```


```bash
virt-install --name aarch64-f32-cdrom --ram 2048 --disk size=10 --os-variant fedora39 --arch aarch64 --cdrom /root/Fedora-Server-netinst-aarch64-39-1.5.iso
```

```bash
wget -O installer-linux http://http.us.debian.org/debian/dists/bullseye/main/installer-arm64/current/images/netboot/debian-installer/arm64/linux
wget -O installer-initrd.gz http://http.us.debian.org/debian/dists/bullseye/main/installer-arm64/current/images/netboot/debian-installer/arm64/initrd.gz

qemu-img create -f qcow2 hda.qcow2 5G

qemu-system-aarch64 -M virt -m 1024 -cpu cortex-a53 -kernel installer-linux -initrd installer-initrd.gz -drive if=none,file=hda.qcow2,format=qcow2,id=hd -device virtio-blk-pci,drive=hd -netdev user,id=mynet -device virtio-net-pci,netdev=mynet -nographic -no-reboot

cp hda.qcow2 debian-golden-image.qcow2
```

```bash
export LIBGUESTFS_BACKEND=direct
virt-filesystems -a hda.qcow2
virt-ls -a hda.qcow2 /boot/
virt-copy-out -a hda.qcow2 /boot/initrd.img-5.10.0-26-arm64 /boot/vmlinuz-5.10.0-26-arm64 .
mkdir -p /virtual-machines/debian
cp initrd.img-5.10.0-26-arm64 /virtual-machines/debian/initrd.img
cp vmlinuz-5.10.0-26-arm64 /virtual-machines/debian/vmlinuz
cp hda.qcow2 /virtual-machines/debian
```

```bash
qemu-system-aarch64 -M virt -m 1024 -cpu cortex-a53 -kernel /virtual-machines/debian/vmlinuz -initrd /virtual-machines/debian/initrd.img -append 'root=/dev/vda2' -drive if=none,file=/virtual-machines/debian/hda.qcow2,format=qcow2,id=hd -device virtio-blk-pci,drive=hd -netdev user,id=mynet -device virtio-net-pci,netdev=mynet -nographic
```

```bash
guestfish --ro -i hda.qcow2
ls /boot
download /boot/initrd.img-5.10.0-26-arm64 initrd.img
```
