# KVM-on-ESXi
To be able to run either Qemu or KVM on top of ESXi, you just need to create a Virtual Machine running Virtual Hardware 10 and enable the (VHV) **Hardware Assisted Virtualization** feature which available in the vSphere Web Client as seen in the screenshot below:

![](https://i.imgur.com/OpJPA9X.png)

## Pre-installation checklist

To run KVM, you need a processor that supports hardware virtualization.
```
root@ubuntu16:~# egrep -c '(vmx|svm)' /proc/cpuinfo
8
```
If **0** it means that your CPU doesn't support hardware virtualization.
If **1** or more it does - but you still need to make sure that virtualization is enabled in the BIOS.

### Use a 64 bit kernel (if possible)
Running a 64 bit kernel on the host operating system is recommended but not required.
- To serve more than 2GB of RAM for your VMs, you must use a 64-bit kernel (see 32bit_and_64bit). On a 32-bit kernel install, you'll be limited to 2GB RAM at maximum for a given VM.
- Also, a 64-bit system can host both 32-bit and 64-bit guests. A 32-bit system can only host 32-bit guests.

To see if your processor is 64-bit, you can run this command:
```
root@ubuntu16:~# egrep -c ' lm ' /proc/cpuinfo
8
```
If **0** is printed, it means that your CPU is not 64-bit.
If **1** or higher, it is. Note: lm stands for Long Mode which equates to a 64-bit CPU.

CPU Checker:
```
apt-get install cpu-checker
```
```
root@ubuntu16:~# kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
```

## Installation
```
root@ubuntu16:~# sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  augeas-lenses binutils build-essential cgmanager cpp cpp-5 cpu-checker dctrl-tools debootstrap devscripts diffstat dpkg-dev dput ebtables fakeroot g++ g++-5 gcc gcc-5 gcc-5-base gettext hardening-includes intltool-debian ipxe-qemu kpartx libaio1 libalgorithm-diff-perl libalgorithm-diff-xs-perl libalgorithm-merge-perl libapt-pkg-perl
  libarchive-zip-perl libasan2 libasound2 libasound2-data libasprintf-dev libasyncns0 libatomic1 libaugeas0 libauthen-sasl-perl libavahi-client3 libavahi-common-data libavahi-common3 libbluetooth3 libboost-iostreams1.58.0 libboost-random1.58.0 libboost-system1.58.0 libboost-thread1.58.0 libbrlapi0.6 libc-dev-bin libc6-dev libcaca0 libcacard0
  libcc1-0 libcgi-fast-perl libcgi-pm-perl libcgmanager0 libcilkrts5 libclass-accessor-perl libclone-perl libcroco3 libdata-alias-perl libdigest-hmac-perl libdistro-info-perl libdpkg-perl libemail-valid-perl libencode-locale-perl libexporter-tiny-perl libfakeroot libfcgi-perl libfdt1 libfile-basedir-perl libfile-fcntllock-perl
  libfile-listing-perl libflac8 libfont-afm-perl libgcc-5-dev libgettextpo-dev libgettextpo0 libgomp1 libhtml-form-perl libhtml-format-perl libhtml-parser-perl libhtml-tagset-perl libhtml-tree-perl libhttp-cookies-perl libhttp-daemon-perl libhttp-date-perl libhttp-message-perl libhttp-negotiate-perl libio-html-perl libio-pty-perl
  libio-socket-inet6-perl libio-socket-ssl-perl libio-string-perl libipc-run-perl libipc-system-simple-perl libiscsi2 libisl15 libitm1 libjpeg-turbo8 libjpeg8 liblist-moreutils-perl liblsan0 liblwp-mediatypes-perl liblwp-protocol-https-perl libmailtools-perl libmpc3 libmpx0 libnet-dns-perl libnet-domain-tld-perl libnet-http-perl
  libnet-ip-perl libnet-smtp-ssl-perl libnet-ssleay-perl libnetcf1 libnih-dbus1 libnl-route-3-200 libnspr4 libnss3 libnss3-nssdb libogg0 libopus0 libparse-debianchangelog-perl libpciaccess0 libperlio-gzip-perl libpixman-1-0 libpulse0 libpython-stdlib libpython2.7-minimal libpython2.7-stdlib libquadmath0 librados2 librbd1 libsdl1.2debian
  libsndfile1 libsocket6-perl libspice-server1 libstdc++-5-dev libstdc++6 libsub-name-perl libtext-levenshtein-perl libtimedate-perl libtsan0 libubsan0 libunistring0 liburi-perl libusbredirparser1 libvirt0 libvorbis0a libvorbisenc2 libwww-perl libwww-robotrules-perl libx86-1 libxen-4.6 libxenstore3.0 libxml2-utils libxslt1.1 libyajl2
  libyaml-libyaml-perl lintian linux-libc-dev make manpages-dev msr-tools patchutils pm-utils python python-cheetah python-libvirt python-minimal python-vm-builder python2.7 python2.7-minimal python3-magic qemu-block-extra qemu-system-common qemu-system-x86 qemu-utils seabios sharutils t1utils unzip vbetool wdiff
Suggested packages:
  augeas-doc binutils-doc cpp-doc gcc-5-locales debtags bsd-mailx | mailx cvs-buildpackage diffoscope devscripts-el dose-extra gnuplot libfile-desktopentry-perl libterm-size-perl libyaml-syck-perl mozilla-devscripts mutt svn-buildpackage w3m debian-keyring equivs libsoap-lite-perl mini-dinstall python-bzrlib g++-multilib g++-5-multilib
  gcc-5-doc libstdc++6-5-dbg gcc-multilib autoconf automake libtool flex bison gdb gcc-doc gcc-5-multilib libgcc1-dbg libgomp1-dbg libitm1-dbg libatomic1-dbg libasan2-dbg liblsan0-dbg libtsan0-dbg libubsan0-dbg libcilkrts5-dbg libmpx0-dbg libquadmath0-dbg gettext-doc autopoint libasound2-plugins alsa-utils augeas-tools libgssapi-perl
  glibc-doc libdata-dump-perl libcrypt-ssleay-perl opus-tools libhtml-template-perl libxml-simple-perl pulseaudio libstdc++-5-doc radvd libauthen-ntlm-perl binutils-multiarch libtext-template-perl make-doc cpufrequtils wireless-tools radeontool python-doc python-tk python-markdown python-pygments python-memcache python2.7-doc binfmt-support
  samba vde2 sgabios ovmf zip
The following NEW packages will be installed:
  augeas-lenses binutils bridge-utils build-essential cgmanager cpp cpp-5 cpu-checker dctrl-tools debootstrap devscripts diffstat dpkg-dev dput ebtables fakeroot g++ g++-5 gcc gcc-5 gettext hardening-includes intltool-debian ipxe-qemu kpartx libaio1 libalgorithm-diff-perl libalgorithm-diff-xs-perl libalgorithm-merge-perl libapt-pkg-perl
  libarchive-zip-perl libasan2 libasound2 libasound2-data libasprintf-dev libasyncns0 libatomic1 libaugeas0 libauthen-sasl-perl libavahi-client3 libavahi-common-data libavahi-common3 libbluetooth3 libboost-iostreams1.58.0 libboost-random1.58.0 libboost-system1.58.0 libboost-thread1.58.0 libbrlapi0.6 libc-dev-bin libc6-dev libcaca0 libcacard0
  libcc1-0 libcgi-fast-perl libcgi-pm-perl libcgmanager0 libcilkrts5 libclass-accessor-perl libclone-perl libcroco3 libdata-alias-perl libdigest-hmac-perl libdistro-info-perl libdpkg-perl libemail-valid-perl libencode-locale-perl libexporter-tiny-perl libfakeroot libfcgi-perl libfdt1 libfile-basedir-perl libfile-fcntllock-perl
  libfile-listing-perl libflac8 libfont-afm-perl libgcc-5-dev libgettextpo-dev libgettextpo0 libgomp1 libhtml-form-perl libhtml-format-perl libhtml-parser-perl libhtml-tagset-perl libhtml-tree-perl libhttp-cookies-perl libhttp-daemon-perl libhttp-date-perl libhttp-message-perl libhttp-negotiate-perl libio-html-perl libio-pty-perl
  libio-socket-inet6-perl libio-socket-ssl-perl libio-string-perl libipc-run-perl libipc-system-simple-perl libiscsi2 libisl15 libitm1 libjpeg-turbo8 libjpeg8 liblist-moreutils-perl liblsan0 liblwp-mediatypes-perl liblwp-protocol-https-perl libmailtools-perl libmpc3 libmpx0 libnet-dns-perl libnet-domain-tld-perl libnet-http-perl
  libnet-ip-perl libnet-smtp-ssl-perl libnet-ssleay-perl libnetcf1 libnih-dbus1 libnl-route-3-200 libnspr4 libnss3 libnss3-nssdb libogg0 libopus0 libparse-debianchangelog-perl libpciaccess0 libperlio-gzip-perl libpixman-1-0 libpulse0 libpython-stdlib libpython2.7-minimal libpython2.7-stdlib libquadmath0 librados2 librbd1 libsdl1.2debian
  libsndfile1 libsocket6-perl libspice-server1 libstdc++-5-dev libsub-name-perl libtext-levenshtein-perl libtimedate-perl libtsan0 libubsan0 libunistring0 liburi-perl libusbredirparser1 libvirt-bin libvirt0 libvorbis0a libvorbisenc2 libwww-perl libwww-robotrules-perl libx86-1 libxen-4.6 libxenstore3.0 libxml2-utils libxslt1.1 libyajl2
  libyaml-libyaml-perl lintian linux-libc-dev make manpages-dev msr-tools patchutils pm-utils python python-cheetah python-libvirt python-minimal python-vm-builder python2.7 python2.7-minimal python3-magic qemu-block-extra qemu-kvm qemu-system-common qemu-system-x86 qemu-utils seabios sharutils t1utils ubuntu-vm-builder unzip vbetool wdiff
The following packages will be upgraded:
  gcc-5-base libstdc++6
2 upgraded, 186 newly installed, 0 to remove and 88 not upgraded.
Need to get 68.4 MB of archives.
After this operation, 265 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
..
..
..
DONE
```

#### Add User to Group
You need to ensure that your username is added to the group libvirtd:
```
root@ubuntu16:~# id -un
root
root@ubuntu16:~# adduser `id -un` libvirtd
Adding user `root' to group `libvirtd' ...
Adding user root to group libvirtd
Done.
```
#### Verify Installation 
```
root@ubuntu16:~# virsh list --all
 Id    Name                           State
----------------------------------------------------

```
#### OPTIONAL: Install the virt-manager
```
root@ubuntu16:~# sudo apt-get install virt-manager

``` 
![](https://i.imgur.com/xfFiZ5u.png)

#### Basic Commands
**virsh**
- Command line interface to all KVM functionality
- Can be used as a “shell”

**virt-manager**


- Graphical interface to most KVM functionality


**virt-install**


- A reasonably good command line installation program


**virt-viewer**


- A consistent method of using VNC for accessing a domain


**qemu-img**


- Used for management of devices on the domain
- virsh will do all the commands this will do

**kvm**


- Used to manage the virtualization interface with Linux



**vncviewer**


- Allows a user VNC access to a domain

### Running First VM using the virt-manager
Copy imnages to the below location:
```
root@ubuntu16:/var/lib/libvirt/images# ls -ltr
total 417460
-rw-rw-r-- 1 medo medo 414187520 Dec 19 13:47 CentOS-6.7-x86_64-minimal.iso
-rw-rw-r-- 1 medo medo  13287936 Dec 19 13:48 cirros-0.3.4-x86_64-disk.img
```
![](https://i.imgur.com/HHPpkHA.png)

Using an ISO file and NATTing as networking:
![](https://i.imgur.com/gTNHEAY.png)

VM up and running..

### Running Second VM using the virt-install

```
root@ubuntu16:~# more mkKVM.sh
#! /usr/bin/env bash


if [[ $# < 1 ]]
then
echo "Usage: $0 image_name [disk_size description image_location os_variant]"
exit 1
fi
## Parameter
INSTANCE=${1}
SIZE=${2:-10}
DESCRIPTION=${3:-"Centos"}
IMAGE_NAME=${4:-"CentOS-6.7-x86_64-minimal.iso"}
OS_VARIANT=${5:-"rhel6"}
KVM_HOME=/var/lib/libvirt/
VIRT_INST=$(which virt-install)
ISO_LOC=/var/lib/libvirt/images
INSTANCE_LOC=/var/lib/libvirt/instance
IMAGE=$ISO_LOC/$IMAGE_NAME


${VIRT_INST} \
--description "$DESCRIPTION" \
--connect=qemu:///system \
--name=$INSTANCE \
--disk size=${SIZE} \
--ram=1024 \
--vcpu=1 \
--graphics=vnc \
--os-type=linux \
--os-variant=$OS_VARIANT \
--network=network=default \
--cdrom=$IMAGE \
--boot=cdrom,hd,menu=on \
--virt-type=kvm

root@ubuntu16:~# ./mkKVM.sh SECOND
```
> The List of Os Variants in KVM
> [http://thomasmullaly.com/2014/11/16/the-list-of-os-variants-in-kvm/](http://thomasmullaly.com/2014/11/16/the-list-of-os-variants-in-kvm/)
> 
### Virtual Machine XML
#### Dump the VM xml configuration

```
root@ubuntu16:~# virsh dumpxml centos
<domain type='kvm'>
  <name>centos</name>
  <uuid>cfd66d81-d1dc-44ab-8631-a56a71c13eaa</uuid>
  <memory unit='KiB'>1048576</memory>
  <currentMemory unit='KiB'>1048576</currentMemory>
  <vcpu placement='static'>1</vcpu>
  <os>
    <type arch='x86_64' machine='pc-i440fx-xenial'>hvm</type>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
  </features>
  <cpu mode='custom' match='exact'>
    <model fallback='allow'>Westmere</model>
  </cpu>
  <clock offset='utc'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <devices>
    <emulator>/usr/bin/kvm-spice</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/var/lib/libvirt/images/centos.qcow2'/>
      <target dev='hda' bus='ide'/>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
    </disk>
    <disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <target dev='hdb' bus='ide'/>
      <readonly/>
      <address type='drive' controller='0' bus='0' target='0' unit='1'/>
    </disk>
    <controller type='usb' index='0' model='ich9-ehci1'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x7'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci1'>
      <master startport='0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0' multifunction='on'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci2'>
      <master startport='2'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x1'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci3'>
      <master startport='4'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x2'/>
    </controller>
    <controller type='pci' index='0' model='pci-root'/>
    <controller type='ide' index='0'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
    </controller>
    <controller type='virtio-serial' index='0'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
    </controller>
    <interface type='direct'>
      <mac address='52:54:00:0d:b8:2c'/>
      <source dev='ens160' mode='bridge'/>
      <model type='rtl8139'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>
    <serial type='pty'>
      <target port='0'/>
    </serial>
    <console type='pty'>
      <target type='serial' port='0'/>
    </console>
    <channel type='spicevmc'>
      <target type='virtio' name='com.redhat.spice.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
    </channel>
    <input type='mouse' bus='ps2'/>
    <input type='keyboard' bus='ps2'/>
    <graphics type='spice' autoport='yes'>
      <image compression='off'/>
    </graphics>
    <sound model='ich6'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </sound>
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </video>
    <redirdev bus='usb' type='spicevmc'>
    </redirdev>
    <redirdev bus='usb' type='spicevmc'>
    </redirdev>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </memballoon>
  </devices>
</domain>

```
#### Modify VM Configuration
```
root@ubuntu16:~# virsh dumpxml centos > centos.xml
root@ubuntu16:~# vim centos.xml
root@ubuntu16:~# virsh define centos.xml
Domain centos defined from centos.xml
```
Changed the memeory from 1024 to 512 
![](https://i.imgur.com/oguB6Oj.png)

#### Using $virsh edit command
```
root@ubuntu16:~# virsh edit centos

Select an editor.  To change later, run 'select-editor'.
  1. /bin/ed
  2. /bin/nano        <---- easiest
  3. /usr/bin/vim.basic
  4. /usr/bin/vim.tiny

Choose 1-4 [2]:

```
### Few virt commands
Shutdown, start or view console
```
$ virsh shutdown centos
$ virsh start centos
$ virt-viewer centos
```

### Clone VM & guestfish

Use the GUI "**virt-manager**" to clone a VM.

What is guestfish?
guestfish allows you to manipulate a clone disk while the disk is off-line. There is no concept of a working (current) directory, thus, no cd command and all paths must be complete. This is very powerful as almost all commands are available including those which resize the file systems. 

```
root@ubuntu16:~# sudo apt-get install libguestfs-tools -y

root@ubuntu16:~# mkdir KVM
root@ubuntu16:~# cd KVM/
root@ubuntu16:~/KVM# mkdir Create
root@ubuntu16:~/KVM# cd Create/
root@ubuntu16:~/KVM/Create# sudo guestfish -d Third -i

Welcome to guestfish, the guest filesystem shell for
editing virtual machine filesystems and disk images.

Type: 'help' for help on commands
      'man' to read the manual
      'quit' to quit the shell

Operating system: CentOS release 6.7 (Final)
/dev/VolGroup/lv_root mounted on /
/dev/sda1 mounted on /boot

><fs> ls /
bin
boot
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
sbin
selinux
srv
sys
tmp
usr
var
><fs> vi  /etc/shadow
><fs>

root@ubuntu16:~/KVM/Create# virsh start Third
Domain Third started

root@ubuntu16:~# virt-viewer Third &
[1] 12620

root@ubuntu16:~# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     Third                          running
 -     centos                         shut off

root@ubuntu16:~# virsh dominfo Third
Id:             3
Name:           Third
UUID:           5d145c80-e59b-455d-8c00-f0e22b0c7ac0
OS Type:        hvm
State:          running
CPU(s):         1
CPU time:       40.1s
Max memory:     524288 KiB
Used memory:    524288 KiB
Persistent:     yes
Autostart:      disable
Managed save:   no
Security model: apparmor
Security DOI:   0
Security label: libvirt-5d145c80-e59b-455d-8c00-f0e22b0c7ac0 (enforcing)

root@ubuntu16:~# virsh autostart Third
Domain Third marked as autostarted

root@ubuntu16:~# virsh dominfo Third
Id:             3
Name:           Third
UUID:           5d145c80-e59b-455d-8c00-f0e22b0c7ac0
OS Type:        hvm
State:          running
CPU(s):         1
CPU time:       40.4s
Max memory:     524288 KiB
Used memory:    524288 KiB
Persistent:     yes
Autostart:      enable
Managed save:   no
Security model: apparmor
Security DOI:   0
Security label: libvirt-5d145c80-e59b-455d-8c00-f0e22b0c7ac0 (enforcing)

root@ubuntu16:~# virsh domstate Third
running

```
> Command $ virsh shutdown <VM> is not working.

In or der to fix, make sure that the acpid is installed and started on the VM:
```
root@ubuntu16:~# ssh  root@192.168.122.3
root@192.168.122.3's password:
Last login: Tue Dec 19 17:43:17 2017 from 192.168.122.1

[root@localhost ~]# yum install acpid
Loaded plugins: fastestmirror
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.coreix.net
 * extras: mirrors.coreix.net
 * updates: mirrors.coreix.net
base                                                                                                                                                                                                                                                         | 3.7 kB     00:00
extras                                                                                                                                                                                                                                                       | 3.4 kB     00:00
updates                                                                                                                                                                                                                                                      | 3.4 kB     00:00
Resolving Dependencies
--> Running transaction check
---> Package acpid.x86_64 0:1.0.10-3.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================================================================================================================================================================
 Package                                                          Arch                                                              Version                                                                   Repository                                                       Size
====================================================================================================================================================================================================================================================================================
Installing:
 acpid                                                            x86_64                                                            1.0.10-3.el6                                                              base                                                             37 k

Transaction Summary
====================================================================================================================================================================================================================================================================================
Install       1 Package(s)

Total download size: 37 k
Installed size: 73 k
Is this ok [y/N]: y
Downloading Packages:
acpid-1.0.10-3.el6.x86_64.rpm                                                                                                                                                                                                                                |  37 kB     00:00
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : acpid-1.0.10-3.el6.x86_64                                                                                                                                                                                                                                        1/1
  Verifying  : acpid-1.0.10-3.el6.x86_64                                                                                                                                                                                                                                        1/1

Installed:
  acpid.x86_64 0:1.0.10-3.el6

Complete!

[root@localhost ~]# /etc/init.d/acpid start
Starting acpi daemon:                                      [  OK  ]
[root@localhost ~]# exit

## Try to shutdown
root@ubuntu16:~# virsh shutdown Third
Domain Third is being shutdown
root@ubuntu16:~# virsh domstate Third
shut off

```
### Using ubuntu-vm-builder


## Networking 
There are a few different ways to allow a virtual machine access to the external network. 
- The default virtual network configuration is known as **Usermode Networking**. NAT is performed on traffic through the host interface to the outside network. 
- Alternatively, you can configure **Bridged Networking** to enable external hosts to directly access services on the guest operating system.

### Bridged Networking

Install the bridge utility
```
sudo apt-get install bridge-utils
```

Edir the interfces file
```
root@ubuntu16:~# vim  /etc/network/interfaces

# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface

## Commented by Moe
##################
#auto ens160
#iface ens160 inet static
#address 192.168.200.2
#netmask 255.255.255.0
#gateway 192.168.200.1
#dns-search vlab.local
#dns-nameservers 192.168.2.62 8.8.8.8


auto br0
iface br0 inet static
        address 192.168.200.2
        network 192.168.200.0
        netmask 255.255.255.0
        broadcast 192.168.200.255
        gateway 192.168.200.1
        dns-nameservers 192.168.2.62 8.8.8.8
        dns-search vlab.local
        bridge_ports ens160
        bridge_stp off
        bridge_fd 0
        bridge_maxwait 0
```
After restart the network:

```
$ sudo /etc/init.d/networking restart

## from the consolde i added a default GW

$ route add default gw 192.168.200.1 br0
```
Change the VM networking from NAT to Bridge 

![](https://i.imgur.com/yhw7wBv.png)

Modify the VM IP information 
```
medo@ubuntu16:~$ ssh root@192.168.200.30
root@192.168.200.30's password:
Last login: Tue Dec 19 18:36:16 2017
[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 52:54:00:5f:d3:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.30/24 brd 192.168.200.255 scope global eth1
    inet6 fe80::5054:ff:fe5f:d311/64 scope link
       valid_lft forever preferred_lft forever
[root@localhost ~]#
[root@localhost ~]# vi  /etc/resolv.conf
[root@localhost ~]#
[root@localhost ~]#
[root@localhost ~]# more /etc/resolv.conf
nameserver 8.8.8.8
[root@localhost ~]# nslookup yahoo.com
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   yahoo.com
Address: 98.138.252.38
Name:   yahoo.com
Address: 206.190.39.42
Name:   yahoo.com
Address: 98.139.180.180

```

Also you can convert an existing VM by editing the XML file 
Replace the below
>     <interface type='network'>
>       <mac address='00:11:22:33:44:55'/>
>       <source network='default'/>
>     </interface>

with
>     <interface type='bridge'>
>       <mac address='00:11:22:33:44:55'/>
>       <source network='br0'/>
>     </interface>

### Private Virtual Switch (Guest-Only Network)
If you want to create a virtual network for use only by the Guests and the host then you must proceed as follows (bellow user$ is the CLI prompt for the user with access to KVM and root# is the CLI prompt for the root user)

Create a bridge device to be used by this network (in the example bellow I'm assigning the IP 10.3.11.254 to the Host):
```
user$ sudo su
root# cat << __EOF__ >> /etc/network/interfaces

#-----------------------
auto privatebr0
iface privatebr0 inet static
        address 10.3.11.254
        netmask 255.255.255.0
        pre-up    brctl addbr privatebr0
        post-down brctl delbr privatebr0
__EOF__
root# /etc/init.d/networking restart
```
Use virsh to create the private network:
```
user$ virsh net-list --all  # check existing KVM networks
Name                 State      Autostart
-----------------------------------------
default              active     yes       

user$ echo '<network> <name>privatenet</name> <bridge name="privatebr0" /> </network>' >> /tmp/net.xml
user$ virsh net-define /tmp/net.xml
Network privatenet defined from /tmp/net.xml

user$ virsh net-list --all
Name                 State      Autostart
-----------------------------------------
default              active     yes       
privatenet           inactive   no        

user$ virsh net-start privatenet
Network privatenet started

user$ virsh net-list --all
Name                 State      Autostart
-----------------------------------------
default              active     yes       
privatenet           active     no        

user$ virsh net-autostart privatenet
Network privatenet marked as autostarted

user$ virsh net-list --all
Name                 State      Autostart
-----------------------------------------
default              active     yes       
privatenet           active     yes       
```
