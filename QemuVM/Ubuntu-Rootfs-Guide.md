# Ubuntu Rootfs from scratch

This guide walks thru the build of a Ubuntu root filesystem from scratch.

This process can be done on a Debian or Ubuntu host.

```bash
# Install pre-reqs on host
sudo apt-get install debootstrap qemu-user-static binfmt-support debian-ports-archive-keyring qemu-system qemu-utils qemu-system-misc

mkdir rootfs-focal

# Generate minimal bootstrap rootfs
sudo debootstrap --arch=ppc64el --variant=minbase focal ./rootfs-focal http://ports.ubuntu.com/ubuntu-ports

# chroot to it. Requires "qemu-user-static qemu-system qemu-utils qemu-system-misc binfmt-support" packages on host
sudo chroot rootfs-focal /bin/bash

# Add package sources
cat >/etc/apt/sources.list <<EOF
deb http://ports.ubuntu.com/ubuntu-ports focal main restricted

deb http://ports.ubuntu.com/ubuntu-ports focal-updates main restricted

deb http://ports.ubuntu.com/ubuntu-ports focal universe
deb http://ports.ubuntu.com/ubuntu-ports focal-updates universe

deb http://ports.ubuntu.com/ubuntu-ports focal multiverse
deb http://ports.ubuntu.com/ubuntu-ports focal-updates multiverse

deb http://ports.ubuntu.com/ubuntu-ports focal-backports main restricted universe multiverse

deb http://ports.ubuntu.com/ubuntu-ports focal-security main restricted
deb http://ports.ubuntu.com/ubuntu-ports focal-security universe
deb http://ports.ubuntu.com/ubuntu-ports focal-security multiverse
EOF

# Install essential packages
apt-get update
apt-get upgrade -y
apt-get install --no-install-recommends -y util-linux haveged openntpd ntpdate openssh-server systemd kmod initramfs-tools conntrack ebtables ethtool iproute2 iptables mount socat ifupdown iputils-ping vim dhcpcd5 neofetch systemd-sysv grub2

# Create base config files
mkdir -p /etc/network
cat >>/etc/network/interfaces <<EOF
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
EOF

cat >/etc/resolv.conf <<EOF
nameserver 1.1.1.1
nameserver 8.8.8.8
EOF

cat >/etc/fstab <<EOF
LABEL=rootfs	/	ext4	user_xattr,errors=remount-ro	0	1
EOF

# Link init to systemd
ln -sf /lib/systemd/systemd /sbin/init

echo "ubuntu-ppc64le" > /etc/hostname

# Disable some services on Qemu
ln -s /dev/null /etc/systemd/network/99-default.link
sed -i 's/^DAEMON_OPTS="/DAEMON_OPTS="-s /' /etc/default/openntpd

# Set root passwd
echo "root:ppc64le" | chpasswd

# Exit chroot
exit

sudo tar -cSf Ubuntu-Focal-rootfs.tar -C rootfs-focal .
bzip2 Ubuntu-Focal-rootfs.tar
rm -rf rootfs-focal
```
