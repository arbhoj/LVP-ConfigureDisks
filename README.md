### Configure LocalVolumeProvisioner Disks for DKP

This repository contains a script that can be used to create disks to be consumed by LocalVolumeProvisioner used by most DKP deployments. It requires a raw disk `>250G` to be attached to all the workder nodes. 

# Pre-requisites:
- This method uses linux logicalvolumes and thus requires the lvm2 module to be installed on the worker nodes where this will be execures
- The commands run here need a user with privilege escalation capability

> Note: The partition postfix for a raw disk (i.e. the dynamically generated postfix for name of the partition created when running fdisk) varies based on the disk type. e.g. for some running fdisk on a disk named `sdh` will create a partition named `sdh1` while some disk types will create a partition named `sdhp1`. This script uses the latter i.e. `p1` as the postfix. Change the value of `PART_POSTFIX` variable to change this behavior.

# Naming Convention

This uses the following naming convention to name the mount directories for different sizes of disks. This is to prevent a very small k8s PVC from binding to a large k8s PV 

> Note: Mount dirs are sorted in an ascending order

10000nnn - Small (11G)
11000nnn - Medium (35G)
11100nnn - Large (105G)


# Steps

1. SSH to the worker node. Create the script and set permission to make it executable 

> Note: Change `PART_POSTFIX` as required. Also Change the number of disks to be created as required. The 

```
cat <<-'EOF' > configure_dkp_disks.sh
#!/bin/bash
##Note This script accepts the name of the raw disk as an input parameter
## e.g. ./configure_dkp_disks.sh sdh
## Pre-req lvm2 module should be installed
#sudo yum install -y lvm2

## Change these as required #########
#This is the name of the logical volume group to be used
VG=dkp

#This is the postfix added to the partion on running fdisk.
#Could be p1 or just 1 depending on the disk
PART_POSTFIX=p1
###################################

RAWDISK=/dev/$1
#Check if disk already formatted and format it if it isn't and create physical volume and volume group for DKP
if test -b "$RAWDISK$PART_POSTFIX";
then
  echo Skipping as disk already formatted
else
  #Create Partition
  (echo o; echo n;echo p;echo 1;echo ; echo ;echo w;) | sudo fdisk $RAWDISK
  sleep 5
  #Create Physical Volume
  sudo pvcreate "$RAWDISK$PART_POSTFIX"
  #Create Virtual Group
  sudo vgcreate $VG "$RAWDISK$PART_POSTFIX"
  sleep 5
#Create logical volumes to be used by k8s PV's
sudo lvcreate -L 11G -n 10000001 $VG
sudo lvcreate -L 11G -n 10000002 $VG
sudo lvcreate -L 11G -n 10000003 $VG
sudo lvcreate -L 11G -n 10000004 $VG
sudo lvcreate -L 35G -n 11000001 $VG
sudo lvcreate -L 35G -n 11000002 $VG
sudo lvcreate -L 105G -n 11100001 $VG

#Make filesystem for logical volumes
sudo mkfs -F -t ext4 /dev/mapper/$VG-10000001
sudo mkfs -F -t ext4 /dev/mapper/$VG-10000002
sudo mkfs -F -t ext4 /dev/mapper/$VG-10000003
sudo mkfs -F -t ext4 /dev/mapper/$VG-10000004
sudo mkfs -F -t ext4 /dev/mapper/$VG-11000001
sudo mkfs -F -t ext4 /dev/mapper/$VG-11000002
sudo mkfs -F -t ext4 /dev/mapper/$VG-11100001

#Make base mount directory under /mnt
sudo mkdir -p /mnt/disks

#Mount 11G volumes
DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-10000001)
sudo mkdir /mnt/disks/10000001
sudo mount -t ext4 /dev/mapper/$VG-10000001 /mnt/disks/10000001
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-10000001` /mnt/disks/10000001 ext4 defaults 0 0 | sudo tee -a /etc/fstab

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-10000002)
sudo mkdir /mnt/disks/10000002
sudo mount -t ext4 /dev/mapper/$VG-10000002 /mnt/disks/10000002
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-10000002` /mnt/disks/10000002 ext4 defaults 0 0 | sudo tee -a /etc/fstab

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-10000003)
sudo mkdir /mnt/disks/10000003
sudo mount -t ext4 /dev/mapper/$VG-10000003 /mnt/disks/10000003
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-10000003` /mnt/disks/10000003 ext4 defaults 0 0 | sudo tee -a /etc/fstab

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-10000004)
sudo mkdir /mnt/disks/10000004
sudo mount -t ext4 /dev/mapper/$VG-10000004 /mnt/disks/10000004
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-10000004` /mnt/disks/10000004 ext4 defaults 0 0 | sudo tee -a /etc/fstab

#Mount 35G volumes
DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11000001)
sudo mkdir /mnt/disks/11000001
sudo mount -t ext4 /dev/mapper/$VG-11000001 /mnt/disks/11000001
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11000001` /mnt/disks/11000001 ext4 defaults 0 0 | sudo tee -a /etc/fstab

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11000002)
sudo mkdir /mnt/disks/11000002
sudo mount -t ext4 /dev/mapper/$VG-11000002 /mnt/disks/11000002
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11000002` /mnt/disks/11000002 ext4 defaults 0 0 | sudo tee -a /etc/fstab

#Mount 105G volumes
DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11100001)
sudo mkdir /mnt/disks/11100001
sudo mount -t ext4 /dev/mapper/$VG-11100001 /mnt/disks/11100001
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11100001` /mnt/disks/11100001 ext4 defaults 0 0 | sudo tee -a /etc/fstab
fi
EOF

chmod +x configure_dkp_disks.sh
```

2. Get the name of the raw disk

```
sudo lsblk
```

3. Execute the script

```
configure_dkp_disks.sh <name of the raw disk>

e.g.:
configure_dkp_disks.sh sdh
```
