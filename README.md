with alpine

```bash
cat /proc/mounts | grep /old_root | awk '{print $2}' | sort -r | while read m; do
  umount -lf "$m"
done

mount --make-rprivate /
vgchange -an -f debian-vg
lvchange -an -f /dev/debian-vg/root



apk add lsblk e2fsprogs wipefs alpine-conf openssh nano sed
sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config

wipefs -a /dev/sda
fdisk /dev/sda
o        # yeni MBR
n        # yeni partition
p        # primary
1        # sda1
ENTER
ENTER
w

mkfs.ext4 /dev/sda1
mount /dev/sda1 /mnt
mkdir -p /mnt/boot

cat > /etc/apk/repositories << EOF
https://dl-cdn.alpinelinux.org/alpine/latest-stable/main
https://dl-cdn.alpinelinux.org/alpine/latest-stable/community
EOF

setup-disk -m sys -k edge /mnt


dd if=/usr/share/syslinux/mbr.bin of=/dev/sda bs=440 count=1
extlinux --install /mnt/boot

3. config var mı kontrol et 
cat /mnt/boot/extlinux.conf



#!/bin/sh
ip a 
ip route
echo "=== Alpine Network Config Script ==="

# Kullanıcıdan bilgiler al
echo -n "Ağ kartı adı (örn: eth0, ens3): "
read IFACE

echo -n "IP adresi (örn: 192.168.1.100): "
read IP

echo -n "Netmask (örn: 255.255.255.0): "
read NETMASK

echo -n "Gateway (örn: 192.168.1.1): "
read GW

# Dosyayı yaz
cat > /mnt/etc/network/interfaces << EOF
auto lo
iface lo inet loopback

auto $IFACE
iface $IFACE inet static
    address $IP
    netmask $NETMASK
    gateway $GW

# fallback eth0
auto eth0
iface eth0 inet static
    address $IP
    netmask $NETMASK
    gateway $GW
EOF

echo "Config yazıldı: /mnt/etc/network/interfaces"

# DNS da ekleyelim
echo "nameserver 1.1.1.1" > /mnt/etc/resolv.conf

echo "DNS ayarlandı (/mnt/etc/resolv.conf)"

echo "Tamamdır 👍"


mount --rbind /dev /mnt/dev
mount -t proc proc /mnt/proc
mount --rbind /sys /mnt/sys
chroot /mnt apk add grub grub-bios openssh
chroot /mnt
apk add ifupdown-ng iproute2
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
echo "nameserver 1.1.1.1" > /etc/resolv.conf
sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
passwd root
rc-update add sshd boot
rc-update add devfs sysinit
rc-update add procfs sysinit
rc-update add sysfs sysinit
rc-update add modules boot

rc-update add hostname boot
rc-update add sshd default
rc-update add networking default


rc-update add root boot
rc-update add devfs sysinit
rc-update add procfs sysinit
rc-update add sysfs sysinit
echo alpine > /etc/hostname
rc-update add hostname boot
cat >> /etc/modules << EOF
virtio_net
e1000
e1000e
EOF
mkdir -p /etc/mkinitfs/features.d

echo "virtio_net" >> /etc/mkinitfs/features.d/network.modules
echo "e1000" >> /etc/mkinitfs/features.d/network.modules
echo "e1000e" >> /etc/mkinitfs/features.d/network.modules
mkinitfs -k $(ls /lib/modules)

mkinitfs -c /etc/mkinitfs/mkinitfs.conf -b / -k $(ls /lib/modules)

grub-mkconfig -o /boot/grub/grub.cfg

cat > /etc/fstab << EOF
/dev/sda1  /  ext4  rw,relatime  0 1
EOF
rc-update add fsck boot

GRUB_CMDLINE_LINUX_DEFAULT="rw quiet modules=virtio_net,e1000,e1000e""
grub-mkconfig -o /boot/grub/grub.cfg
sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config

mkdir -p /etc/local.d

cat > /etc/local.d/network.start << 'EOF'
sleep 2
ip link set eth0 up
ip addr add 192.168.1.100/24 dev eth0
ip route add default via 192.168.1.1
EOF

chmod +x /etc/local.d/network.start
rc-update add local default


```
