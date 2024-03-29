```bash
cat <<-'EOF' > configure_small_disks.sh
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
PART_POSTFIX=1
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
sudo lvcreate -L 1G -n 10000001 $VG
sudo lvcreate -L 1G -n 10000002 $VG
sudo lvcreate -L 1G -n 10000003 $VG
sudo lvcreate -L 1G -n 10000004 $VG
sudo lvcreate -L 1G -n 10000005 $VG
sudo lvcreate -L 2G -n 11000001 $VG
sudo lvcreate -L 2G -n 11000002 $VG
sudo lvcreate -L 2G -n 11000003 $VG
sudo lvcreate -L 2G -n 11000004 $VG
sudo lvcreate -L 2G -n 11000005 $VG
sudo lvcreate -L 5G -n 11100001 $VG
sudo lvcreate -L 5G -n 11100002 $VG
sudo lvcreate -L 11G -n 11110001 $VG
sudo lvcreate -L 11G -n 11110002 $VG

#Make filesystem for logical volumes
sudo mkfs -F -t ext4 /dev/mapper/$VG-10000001
sudo mkfs -F -t ext4 /dev/mapper/$VG-10000002
sudo mkfs -F -t ext4 /dev/mapper/$VG-10000003
sudo mkfs -F -t ext4 /dev/mapper/$VG-10000004
sudo mkfs -F -t ext4 /dev/mapper/$VG-10000005
sudo mkfs -F -t ext4 /dev/mapper/$VG-11000001
sudo mkfs -F -t ext4 /dev/mapper/$VG-11000002
sudo mkfs -F -t ext4 /dev/mapper/$VG-11000003
sudo mkfs -F -t ext4 /dev/mapper/$VG-11000004
sudo mkfs -F -t ext4 /dev/mapper/$VG-11000005
sudo mkfs -F -t ext4 /dev/mapper/$VG-11100001
sudo mkfs -F -t ext4 /dev/mapper/$VG-11100002
sudo mkfs -F -t ext4 /dev/mapper/$VG-11110001
sudo mkfs -F -t ext4 /dev/mapper/$VG-11110002

#Make base mount directory under /mnt
sudo mkdir -p /mnt/disks

#Mount 1G volumes
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

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-10000005)
sudo mkdir /mnt/disks/10000005
sudo mount -t ext4 /dev/mapper/$VG-10000005 /mnt/disks/10000005
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-10000005` /mnt/disks/10000005 ext4 defaults 0 0 | sudo tee -a /etc/fstab

#Mount 2G volumes
DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11000001)
sudo mkdir /mnt/disks/11000001
sudo mount -t ext4 /dev/mapper/$VG-11000001 /mnt/disks/11000001
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11000001` /mnt/disks/11000001 ext4 defaults 0 0 | sudo tee -a /etc/fstab

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11000002)
sudo mkdir /mnt/disks/11000002
sudo mount -t ext4 /dev/mapper/$VG-11000002 /mnt/disks/11000002
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11000002` /mnt/disks/11000002 ext4 defaults 0 0 | sudo tee -a /etc/fstab

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11000003)
sudo mkdir /mnt/disks/11000003
sudo mount -t ext4 /dev/mapper/$VG-11000003 /mnt/disks/11000003
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11000003` /mnt/disks/11000003 ext4 defaults 0 0 | sudo tee -a /etc/fstab

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11000004)
sudo mkdir /mnt/disks/11000004
sudo mount -t ext4 /dev/mapper/$VG-11000004 /mnt/disks/11000004
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11000004` /mnt/disks/11000004 ext4 defaults 0 0 | sudo tee -a /etc/fstab

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11000005)
sudo mkdir /mnt/disks/11000005
sudo mount -t ext4 /dev/mapper/$VG-11000005 /mnt/disks/11000005
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11000005` /mnt/disks/11000005 ext4 defaults 0 0 | sudo tee -a /etc/fstab

#Mount 5G volumes
DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11100001)
sudo mkdir /mnt/disks/11100001
sudo mount -t ext4 /dev/mapper/$VG-11100001 /mnt/disks/11100001
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11100001` /mnt/disks/11100001 ext4 defaults 0 0 | sudo tee -a /etc/fstab

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11100002)
sudo mkdir /mnt/disks/11100002
sudo mount -t ext4 /dev/mapper/$VG-11100002 /mnt/disks/11100002
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11100002` /mnt/disks/11100002 ext4 defaults 0 0 | sudo tee -a /etc/fstab

#Mount 11G volumes
DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11110001)
sudo mkdir /mnt/disks/11110001
sudo mount -t ext4 /dev/mapper/$VG-11110001 /mnt/disks/11110001
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11110001` /mnt/disks/11110001 ext4 defaults 0 0 | sudo tee -a /etc/fstab

DISK_UUID=$(sudo blkid -s UUID -o value blkid /dev/mapper/$VG-11110002)
sudo mkdir /mnt/disks/11110002
sudo mount -t ext4 /dev/mapper/$VG-11110002 /mnt/disks/11110002
echo UUID=`sudo blkid -s UUID -o value /dev/mapper/$VG-11110002` /mnt/disks/11110002 ext4 defaults 0 0 | sudo tee -a /etc/fstab
fi
EOF

chmod +x configure_small_disks.sh
```
