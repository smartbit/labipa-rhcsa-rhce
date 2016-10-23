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

Linux Foundation
[Handbook](http://training.linuxfoundation.org/go/candidate_handbook)

## RHCSE
[RHCSE skills-assessment](http://www.redhat.com/en/services/training/skills-assessment)

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
