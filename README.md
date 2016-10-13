## Install instructions Lab Environment _Red Hat RHCSA-RHCE 7 Cert Guide_

The Linux exams of Linux Foundation and RedHat are performance based. To pass you **must** practise. Sander van Vugt offers an lab environment created with VMware fusion. He offers two versions
- V2.0, available for download at http://rhatcert.com or through the book’s website at [http://pearsonitcertification.com](http://www.pearsonitcertification.com/promotions/book-registration-red-hat-rhcsa-rhce-7-cert-guide-premium-140923#vm).
- V3.0 is available through http://rhatcert.com only.
- You can also create the lab environment from scratch with Vagrant and Ansible [RHCSA RHCE Lab environment](../../../RHCSA-RHCE-Lab-Environment/tree/eth1).

You can run these lab environments in VMware Fusion/Workstation, in virtualBox or on Linux import them in LVM.



|  | Pro  | Con  |
|---          |---|---|
| [V2.0](http://www.rhatcert.com/downloads/)  |1) has CentoS 7.1 similar to some exams, <br />eg nmcli v 0.9| 1) yum update takes long & probably fails  <br />2) needs little tweaking for alls labs to work  |
| [V3.0](http://www.rhatcert.com/downloads/)  |1) updates from private repository 1511<br />2) Sander recommends this version | 1) you'd never practised with 7.0.1406  <br />2) needs little tweaking for alls labs to work|
| [Vagrant](../../../RHCSA-RHCE-Lab-Environment/)  | 1) always latest CentoS version, <br />updates to 7.3 automatically<br >2) works out of the box<br />3) could be adapted for [certdepot](http://certdepot.com) |  1) you'll probably not see this RHEL<br > version on the exam<br />2) no support for Virtualization labs<br /> 3) you'd never practised with 7.0.1406|

|   | vmware | virtualbox  |  libvirt | any Cloud |
|---|---|---|---|---|
|| - macOS $/€ 82<br />- Windows $250 | free | comes with<br />Linux|- works on<br />any ssh client<br />- recurring $€|
| [V2.0](http://www.rhatcert.com/downloads/)  | - supported<br />- supports LVM  | - works<br />- no LVM  | - should work <br >- supports LMV   | - not tested  |
|  [V3.0](http://www.rhatcert.com/downloads/) | - supported<br />- supports LVM   |   | - should work <br >- supports LVM  | - not tested  |
|  [Vagrant](../../../RHCSA-RHCE-Lab-Environment/) | - [not<br /> recommended](http://blog.scottlowe.org/2016/09/28/why-now-using-virtualbox-with-vagrant/)<br />- [$$](https://www.vagrantup.com/vmware/#buy-now) |   | - not tested<br >- should work <br >- supports vmx  | - not tested  |




## Pro Tip
On **macOS** switch from VirtualBox to VMware Fusion by following these steps<br />- shutdown virtualbox<br />- execute:
```bash
netstat -rn | grep -e 192.168.4 -e Destination
sudo route -n delete 192.168.4.0/24
open -a VMware\ Fusion; sleep 5
netstat -rn | grep -e 192.168.4 -e Destination
route get 192.168.4.0
```

On **macOS** switch from VMware Fusion to VirtualBox:<br />- shutdown VMware Fusion<br />- execute:
```bash
netstat -rn | grep -e 192.168.4 -e Destination
LABIPAVBOXNET=`ifconfig | pcregrep -M -o '^[^\t:]+:([^\n]|\n\t)*192.168.4' | egrep -o -m 1 '^[^:]+'`
echo $LABIPAVBOXNET
sudo route -n add 192.168.4.0/24 -interface $LABIPAVBOXNET
netstat -rn | grep -e 192.168.4 -e Destination
route get 192.168.4.0
ssh-keygen -R 192.168.4.200; ssh-keygen -R 192.168.4.210; ssh-keygen -R 192.168.4.220
vagrant status
```

Importing this in VirtualBox on macOS is not a trivial case as VMware Fusion uses different network settings. This guide explains how to import Sander's Lab and get a working environment (and maybe learn to understand `nmcli` better).



# Create a host-only network
Open the setting of VirtualBox, go to Network, create an new Host-only Network ![create host-only network](https://cloud.githubusercontent.com/assets/16225624/18674769/a0e60e08-7f50-11e6-9518-a41f4e08bad6.png)
<br />and assign it an network `192.168.4.0/24` with an host ip eg `192.168.4.4`
![subnet192 168 4](https://cloud.githubusercontent.com/assets/16225624/18674930/0fe6f42a-7f51-11e6-9da0-4bd75428a341.png)

## Import _Server1_ into VirtualBox
Download the ovf v2.0 and extract the zip file.

Import the each of the ovf files in VirtualBox
![import labipa server1](https://cloud.githubusercontent.com/assets/16225624/18675014/5266025a-7f51-11e6-96db-aebb874096b7.png)


Before pressing Import, rename the VM and remove dvd & usb ![rename vm and remove dvd usb](https://cloud.githubusercontent.com/assets/16225624/18675161/cb83c7bc-7f51-11e6-8871-d0e84ae8ce4a.png)
<br/>On the FreeIPA server update memory to 2GB.

Before starting the VM, edit the Settings ![edit vm](https://cloud.githubusercontent.com/assets/16225624/18677881/9d7f3fdc-7f5a-11e6-9c87-d6e1cc06352c.png)

Add an host adapter and connect it to host-only network `192.168.4.0/24` ![add host adapter and connect to host-only network](https://cloud.githubusercontent.com/assets/16225624/18675323/4a892ca0-7f52-11e6-8783-0ec111df9d2b.png)

The VM now has 2 adapters:
- Adapter 2 connected to eg. `enp0s8`/`eth0` on subnet 192.168.4.0/24 and will **not** be the default gateway.
- Adapter 1 connected to eg. `enp0s17`/`eth1` will be NAT for internet access (eg 10.0.2.2/24) but will **not** be setting DNS

<!-- ![2 adapters](https://cloud.githubusercontent.com/assets/16225624/18675550/fd218998-7f52-11e6-90ed-e40c4724fdd4.png) -->

Then start the new vm ![start server1](https://cloud.githubusercontent.com/assets/16225624/18676011/8b31155e-7f54-11e6-86ee-d4c739acd62b.png) In Terminal connect with ssh and excute these commands for Server1:
```bash
ssh root@192.168.4.210
cat /etc/centos-release; nmcli --version
mkdir /etc/openldap/cacerts # make the lab easier ;-)
nmcli connection add con-name eth1 type ethernet ifname enp0s17 autoconnect yes save yes
nmcli connection modify eth1 ipv4.ignore-auto-dns yes
nmcli device disconnect enp0s17;nmcli connection up eth1
cat /etc/resolv.conf
ip r
# RHEL 7.0 uses NetworkManager v0.9.9.1 and doesn't support ipv4.gateway, therefor use ipv4.addresses without 'gw'
nmcli connection modify eth0 ipv4.never-default yes ipv6.never-default yes ipv4.addresses "192.168.4.210/24"
systemctl restart NetworkManager; systemctl restart NetworkManager
ip r
# systemctl set-default graphical.target # optional
yum -y update --security ; shutdown now
```
![untitled 23](https://cloud.githubusercontent.com/assets/16225624/18674509/c3c944ea-7f4f-11e6-9a2b-967423186654.png)

To save the current state create a snapshot ![take snapshot copy](https://cloud.githubusercontent.com/assets/16225624/18677143/213924d0-7f58-11e6-9eb1-f9f164a6b16c.png)
<!-- ![take snapshot](https://cloud.githubusercontent.com/assets/16225624/18676516/25bcaac4-7f56-11e6-9ef8-ed19e4b13a61.png) -->

Your server1 is ready for exercises ![snapshot created](https://cloud.githubusercontent.com/assets/16225624/18674637/292f860a-7f50-11e6-9325-848844b7b7a6.png)

## FreeIPA
Like Server1, with these commands
```bash
ssh root@192.168.4.220
cat /etc/centos-release; nmcli --version
grep -B2 -A1 8.8.8.8 /etc/named.conf
ip r
# RHEL 7.1+ uses NetworkManager v1.0.0 which does support ipv4.gateway
nmcli connection modify eth0 ipv4.never-default yes ipv6.never-default yes ipv4.gateway "" ipv4.dns 192.168.4.200
ip r del default via 192.168.4.2 dev eth0
nmcli connection add con-name eth1 type ethernet ifname eth1 save yes autoconnect no
nmcli connection modify eth1 ipv4.ignore-auto-dns yes ipv6.ignore-auto-dns yes connection.autoconnect yes
sleep 2
ip route add default via 10.0.2.2 dev eth1
ip r
echo $'password\npassword\npassword' | kinit admin # password expired
klist
yum -y update --security  ; shutdown now
```


## Server2
Like Server1, with these commands
```bash
ssh root@192.168.4.220
cat /etc/centos-release; nmcli --version
mkdir /etc/openldap/cacerts
nmcli connection add con-name eth1 type ethernet ifname eth1 autoconnect yes save yes
nmcli connection modify eth1 ipv4.ignore-auto-dns yes
nmcli device disconnect eth1;nmcli connection up eth1
cat /etc/resolv.conf
ip r
nmcli connection modify eth0 ipv4.never-default yes ipv6.never-default yes ipv4.addresses "192.168.4.220/24"
systemctl restart NetworkManager; systemctl restart NetworkManager
ip r
yum -y update --security ; shutdown now
```
Alternatively
```bash
ssh root@192.168.4.220
cat /etc/centos-release; nmcli --version
nmcli connection add con-name eth1 type ethernet ifname eth1 save yes autoconnect no
nmcli connection modify eth1 ipv4.ignore-auto-dns yes connection.autoconnect yes
sleep 1
nmcli connection modify eth0 ipv4.never-default yes ipv6.never-default yes ipv4.addresses "192.168.4.220/24"
sleep 1
ip r del default via 192.168.4.2 dev eth0
sleep 1
ip r
cat /etc/resolv.conf # yum only works if FreeIPA @ 192.168.4.200 is up and serving DNS
# 'yum -y groupinstall Directory\ Client' fails on server2
yum -y update --security ; shutdown now
```
