# iscsi# Configure Target

---

1. Ensure that host has access to the repositories to install the required packages.
2. Host has network configuration.
3. Host has name resolution working properly either by using DNS or /etc/hosts file.
4. You have a storage device present which you want to share.
5. You have two machines to test this. 1st machine will act as Target (Iscsi server) and 2nd machine will act as initiator(iscsi client)

---
- First insatll the target packages
```bash
dnf install targetcli -y
```
- Enable the service
```bash
systemctl enable --now target
```
- We will use **nvme0n2** device as iscsi Lun
```bash
# lsblk 
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0  30G  0 disk 
├─nvme0n1p1   259:1    0   1G  0 part /boot
└─nvme0n1p2   259:2    0  29G  0 part 
  ├─rhel-root 253:0    0  26G  0 lvm  /
  └─rhel-swap 253:1    0   3G  0 lvm  [SWAP]
nvme0n2       259:3    0  80G  0 disk 
```
```bash
targetcli
/> /backstores/block create name=iscsi_disk dev=/dev/nvme0n2
Created block storage object iscsi_disk using /dev/nvme0n2.
/> exit
```
Create the target and Lun
```bash
targetcli /iscsi create iqn.2025-05.com.lab.example:iscsi
targetcli /iscsi/iqn.2025-05.com.lab.example:iscsi/tpg1/luns create /backstores/block/iscsi_disk
```
Create the ACL to allow intiator to access the LUN
```bash
targetcli /iscsi/iqn.2025-05.com.lab.example:iscsi/tpg1/acls create iqn.2025-05.com.example.lab:initiator01
Created Node ACL for iqn.2025-05.com.example.lab:initiator01
Created mapped LUN 0.
```
- To list Luns and ACLs run below command
```bash
 targetcli ls
```
Disable firewalld and selinux
```bash
systemctl disable firewalld --now
setenforce 0
```
## Steps for inititate
- Install the packages
```bash
dnf install iscsi-initiator-utils -y
```
- Start and Enable and service
```bash
systemctl enable --now iscsid
systemctl enable --now iscsi
```
- Configure the iqn
```bash
cat /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2025-05.com.example.lab:initiator01
```
- Perform Iscsi Discovery
```bash
iscsiadm -m discovery -t sendtargets -p iscsi.lab.example.com
```
- Now login to the iscsi Lun
```bash
iscsiadm -m node -T iqn.2025-05.com.lab.example:iscsi -p iscsi.lab.example.com -l
```
- After successfull login Check the device in lsblk
```bash
lsblk
```
