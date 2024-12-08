# TP - 2 - Modest Storage SAN 

## II. San Network

### 1. Storage machines

#### A. Disks and RAID.

Nos raids sont prêts à l'emploi : 
````
[mmederic@sto1 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   20G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0   19G  0 part  
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
sdb           8:16   0    5G  0 disk  
└─md0         9:0    0    5G  0 raid1 
sdc           8:32   0    5G  0 disk  
└─md1         9:1    0    5G  0 raid1 
sdd           8:48   0    5G  0 disk  
└─md2         9:2    0    5G  0 raid1 
sde           8:64   0    5G  0 disk  
└─md1         9:1    0    5G  0 raid1 
sdf           8:80   0    5G  0 disk  
└─md2         9:2    0    5G  0 raid1 
sdg           8:96   0    5G  0 disk  
└─md0         9:0    0    5G  0 raid1 
sr0          11:0    1 1024M  0 rom
````

Voici sto2 : 
````
[mmederic@sto2 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   20G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0   19G  0 part  
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
sdb           8:16   0    5G  0 disk  
└─md0         9:0    0    5G  0 raid1 
sdc           8:32   0    5G  0 disk  
└─md0         9:0    0    5G  0 raid1  
sdd           8:48   0    5G  0 disk  
└─md1         9:1    0    5G  0 raid1 
sde           8:64   0    5G  0 disk  
└─md1         9:1    0    5G  0 raid1 
sdf           8:80   0    5G  0 disk  
└─md2         9:2    0    5G  0 raid1 
sdg           8:96   0    5G  0 disk  
└─md2         9:2    0    5G  0 raid1 
sr0          11:0    1 1024M  0 rom  
````

#### B. iSCSi target 

On a installé et démarré le service : 
````
[mmederic@sto2 ~]$ sudo systemctl status target
● target.service - Restore LIO kernel target configuration
     Loaded: loaded (/usr/lib/systemd/system/target.service; enabled; preset: disabled)
     Active: active (exited) since Sun 2024-12-08 17:48:53 CET; 5min ago
    Process: 787 ExecStart=/usr/bin/targetctl restore (code=exited, status=0/SUCCESS)
   Main PID: 787 (code=exited, status=0/SUCCESS)
        CPU: 184ms

Dec 08 17:48:52 sto2.tp2 systemd[1]: Starting Restore LIO kernel target configuration...
Dec 08 17:48:53 sto2.tp2 systemd[1]: Finished Restore LIO kernel target configuration.
````

Notre targetcli :
````
[mmederic@sto2 ~]$ sudo targetcli
targetcli shell version 2.1.57
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> ls
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 0]
  | o- fileio ................................................................................................. [Storage Objects: 3]
  | | o- data-chunk1 ...................................................................... [/dev/md0 (5.0GiB) write-back activated]
  | | | o- alua ................................................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | | o- data-chunk2 ...................................................................... [/dev/md1 (5.0GiB) write-back activated]
  | | | o- alua ................................................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | | o- data-chunk3 ...................................................................... [/dev/md2 (5.0GiB) write-back activated]
  | |   o- alua ................................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 3]
  | o- iqn.2024-12.tp2.b3:data-chunk1 .................................................................................... [TPGs: 1]
  | | o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  | |   o- acls .......................................................................................................... [ACLs: 1]
  | |   | o- iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator ...................................................... [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .......................................................................... [lun0 fileio/data-chunk1 (rw)]
  | |   o- luns .......................................................................................................... [LUNs: 1]
  | |   | o- lun0 ............................................................... [fileio/data-chunk1 (/dev/md0) (default_tg_pt_gp)]
  | |   o- portals .................................................................................................... [Portals: 1]
  | |     o- 0.0.0.0:3260 ..................................................................................................... [OK]
  | o- iqn.2024-12.tp2.b3:data-chunk2 .................................................................................... [TPGs: 1]
  | | o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  | |   o- acls .......................................................................................................... [ACLs: 1]
  | |   | o- iqn.2024-12.tp2.b3:data-chunk2:chunk2-initiator ...................................................... [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .......................................................................... [lun0 fileio/data-chunk2 (rw)]
  | |   o- luns .......................................................................................................... [LUNs: 1]
  | |   | o- lun0 ............................................................... [fileio/data-chunk2 (/dev/md1) (default_tg_pt_gp)]
  | |   o- portals .................................................................................................... [Portals: 1]
  | |     o- 0.0.0.0:3260 ..................................................................................................... [OK]
  | o- iqn.2024-12.tp2.b3:data-chunk3 .................................................................................... [TPGs: 1]
  |   o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  |     o- acls .......................................................................................................... [ACLs: 1]
  |     | o- iqn.2024-12.tp2.b3:data-chunk3:chunk3-initiator ...................................................... [Mapped LUNs: 1]
  |     |   o- mapped_lun0 .......................................................................... [lun0 fileio/data-chunk3 (rw)]
  |     o- luns .......................................................................................................... [LUNs: 1]
  |     | o- lun0 ............................................................... [fileio/data-chunk3 (/dev/md2) (default_tg_pt_gp)]
  |     o- portals .................................................................................................... [Portals: 1]
  |       o- 0.0.0.0:3260 ..................................................................................................... [OK]
  o- loopback ......................................................................................................... [Targets: 0]
````

### 2. Chunks machine

#### A. Simple iSCSi 

Notre iscsi service est bien installé et lancé : 
````
[mmederic@chunk1 ~]$ systemctl status iscsi
● iscsi.service - Login and scanning of iSCSI devices
     Loaded: loaded (/usr/lib/systemd/system/iscsi.service; indirect; preset: enabled)
     Active: active (exited) since Sun 2024-12-08 17:47:57 CET; 9min ago
       Docs: man:iscsiadm(8)
             man:iscsid(8)
    Process: 841 ExecStart=/usr/sbin/iscsiadm -m node --loginall=automatic -W (code=exited, status=0/SUCCESS)
    Process: 850 ExecReload=/usr/sbin/iscsiadm -m node --loginall=automatic -W (code=exited, status=0/SUCCESS)
   Main PID: 841 (code=exited, status=0/SUCCESS)
        CPU: 7ms
````

On a configuré nos initiator, puis une ptite liste de nos discoveries : 
````
[mmederic@chunk1 ~]$ sudo iscsiadm -m node
[sudo] password for mmederic: 
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.1.2:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.2.1:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.2.2:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.1.2:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.2.1:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.2.2:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk3
10.3.1.2:3260,1 iqn.2024-12.tp2.b3:data-chunk3
10.3.2.1:3260,1 iqn.2024-12.tp2.b3:data-chunk3
10.3.2.2:3260,1 iqn.2024-12.tp2.b3:data-chunk3
````

On a modifié la conf iscsi : 
````
[mmederic@chunk1 ~]$ sudo cat /etc/iscsi/iscsid.conf | grep node.session.timeo.replacement_timeout
node.session.timeo.replacement_timeout = 0
````

On a bien nos 4 volumes : 
````
[mmederic@chunk1 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   20G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0   19G  0 part  
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
sdb           8:16   0    5G  0 disk  
sdc           8:32   0    5G  0 disk  
sdd           8:48   0    5G  0 disk  
sde           8:64   0    5G  0 disk  
sr0          11:0    1 1024M  0 rom   
````

Voilà notre commande `iscsiadm -m session -P 3` : 
````
[mmederic@chunk1 ~]$ sudo iscsiadm -m session -P 3 
iSCSI Transport Class version 2.0-870
version 6.2.1.9
Target: iqn.2024-12.tp2.b3:data-chunk1 (non-flash)
        Current Portal: 10.3.2.2:3260,1
        Persistent Portal: 10.3.2.2:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.2.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 1
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 3  State: running
                scsi3 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdb          State: running
        Current Portal: 10.3.1.2:3260,1
        Persistent Portal: 10.3.1.2:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.1.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 10
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 10 State: running
                scsi10 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdd          State: running
        Current Portal: 10.3.1.1:3260,1
        Persistent Portal: 10.3.1.1:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.1.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 4
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 4  State: running
                scsi4 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdc          State: running
        Current Portal: 10.3.2.1:3260,1
        Persistent Portal: 10.3.2.1:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
                Iface IPaddress: 10.3.2.101
                Iface HWaddress: default
                Iface Netdev: default
                SID: 5
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 5  State: running
                scsi5 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sde          State: running
````

#### B. Multipathing 

**insert Lilou moultipass meme**

*prévisible je sais*


Multipath déjà installé : 
````
[mmederic@chunk1 ~]$ systemctl status multipathd
● multipathd.service - Device-Mapper Multipath Device Controller
     Loaded: loaded (/usr/lib/systemd/system/multipathd.service; enabled; preset: enabled)
     Active: active (running) since Sun 2024-12-08 17:47:54 CET; 22min ago
TriggeredBy: ● multipathd.socket
    Process: 640 ExecStartPre=/sbin/modprobe -a scsi_dh_alua scsi_dh_emc scsi_dh_rdac dm-multipath (code=exited, status=0/SUCCESS)
    Process: 643 ExecStartPre=/sbin/multipath -A (code=exited, status=0/SUCCESS)
   Main PID: 644 (multipathd)
     Status: "up"
      Tasks: 7
     Memory: 21.0M
        CPU: 433ms
     CGroup: /system.slice/multipathd.service
             └─644 /sbin/multipathd -d -s
````

Notre fichier de multipath.conf : 
````
[mmederic@chunk1 ~]$ cat /etc/multipath.conf 
# This is a basic configuration file with some examples, for device mapper
# multipath.
#
# For a complete list of the default configuration values, run either
# multipath -t
# or
# multipathd show config
#
# For a list of configuration options with descriptions, see the multipath.conf
# man page

## By default, devices with vendor = "IBM" and product = "S/390.*" are
## blacklisted. To enable mulitpathing on these devies, uncomment the
## following lines.
#blacklist_exceptions {
#       device {
#               vendor  "IBM"
#               product "S/390.*"
#       }
#}

## Use user friendly names, instead of using WWIDs as names.
defaults {
  user_friendly_names yes
  find_multipaths yes
  path_grouping_policy failover
  features "1 queue_if_no_path"
  no_path_retry 100

}
````

Notre lsblk multipath :
````
[mmederic@chunk1 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   20G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0   19G  0 part  
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
sdb           8:16   0    5G  0 disk  
└─mpatha    253:2    0    5G  0 mpath 
sdc           8:32   0    5G  0 disk  
└─mpathb    253:4    0    5G  0 mpath 
sdd           8:48   0    5G  0 disk  
└─mpatha    253:2    0    5G  0 mpath 
sde           8:64   0    5G  0 disk  
└─mpathb    253:4    0    5G  0 mpath 
sr0          11:0    1 1024M  0 rom  

[mmederic@chunk1 ~]$ sudo multipath -ll
mpatha (36001405e5bf5c158c4e4a5398ccba2cc) dm-2 LIO-ORG,data-chunk1
size=5.0G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=enabled
| `- 3:0:0:0  sdb 8:16 active ready running
`-+- policy='service-time 0' prio=50 status=active
  `- 10:0:0:0 sdd 8:48 active ready running
mpathb (3600140566ee7fadd60740a0afd27f8d5) dm-4 LIO-ORG,data-chunk1
size=5.0G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 4:0:0:0  sdc 8:32 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 5:0:0:0  sde 8:64 active ready running
````

### 3. Formatage et montage : 

Petit fdisk -l : 
````
[mmederic@chunk1 ~]$ sudo !!
sudo fdisk -l
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x431a3bfd

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 41943039 39843840  19G 8e Linux LVM


Disk /dev/mapper/rl-root: 17 GiB, 18249416704 bytes, 35643392 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rl-swap: 2 GiB, 2147483648 bytes, 4194304 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 5 GiB, 5363466240 bytes, 10475520 sectors
Disk model: data-chunk1     
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 8388608 bytes
Disklabel type: dos
Disk identifier: 0x9dadc42a

Device     Boot Start      End  Sectors Size Id Type
/dev/sdb1       16384 10475519 10459136   5G 83 Linux


Disk /dev/mapper/mpatha: 5 GiB, 5363466240 bytes, 10475520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 8388608 bytes
Disklabel type: dos
Disk identifier: 0x9dadc42a

Device              Boot Start      End  Sectors Size Id Type
/dev/mapper/mpatha1      16384 10475519 10459136   5G 83 Linux


Disk /dev/sdc: 5 GiB, 5363466240 bytes, 10475520 sectors
Disk model: data-chunk1     
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 8388608 bytes
Disklabel type: dos
Disk identifier: 0x9593d560

Device     Boot Start      End  Sectors Size Id Type
/dev/sdc1       16384 10475519 10459136   5G 83 Linux


Disk /dev/sdd: 5 GiB, 5363466240 bytes, 10475520 sectors
Disk model: data-chunk1     
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 8388608 bytes
Disklabel type: dos
Disk identifier: 0x9dadc42a

Device     Boot Start      End  Sectors Size Id Type
/dev/sdd1       16384 10475519 10459136   5G 83 Linux


Disk /dev/sde: 5 GiB, 5363466240 bytes, 10475520 sectors
Disk model: data-chunk1     
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 8388608 bytes
Disklabel type: dos
Disk identifier: 0x9593d560

Device     Boot Start      End  Sectors Size Id Type
/dev/sde1       16384 10475519 10459136   5G 83 Linux


Disk /dev/mapper/mpathb: 5 GiB, 5363466240 bytes, 10475520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 8388608 bytes
Disklabel type: dos
Disk identifier: 0x9593d560

Device              Boot Start      End  Sectors Size Id Type
/dev/mapper/mpathb1      16384 10475519 10459136   5G 83 Linux
````

On en est là : 
````
[mmederic@chunk1 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   20G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0   19G  0 part  
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
sdb           8:16   0    5G  0 disk  
└─mpatha    253:2    0    5G  0 mpath 
  └─mpatha1 253:3    0    5G  0 part  /mnt/data_chunk1
sdc           8:32   0    5G  0 disk  
└─mpathb    253:4    0    5G  0 mpath 
  └─mpathb1 253:5    0    5G  0 part  /mnt/data_chunk2
sdd           8:48   0    5G  0 disk  
└─mpatha    253:2    0    5G  0 mpath 
  └─mpatha1 253:3    0    5G  0 part  /mnt/data_chunk1
sde           8:64   0    5G  0 disk  
└─mpathb    253:4    0    5G  0 mpath 
  └─mpathb1 253:5    0    5G  0 part  /mnt/data_chunk2
sr0          11:0    1 1024M  0 rom   
[mmederic@chunk1 ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                229M     0  229M   0% /dev/shm
tmpfs                 92M  2.5M   89M   3% /run
/dev/mapper/rl-root   17G  1.7G   16G  11% /
/dev/sda1            960M  431M  530M  45% /boot
/dev/mapper/mpatha1  5.0G   68M  4.9G   2% /mnt/data_chunk1
/dev/mapper/mpathb1  5.0G   68M  4.9G   2% /mnt/data_chunk2
tmpfs                 46M     0   46M   0% /run/user/1000
````

## 4. Test. 

aïe ouïlle j'ai rien réussie à faire avec ma machine, 
mon gns3 trop boggé éteint ne laisse plus passer de connexions de temps à autres, 
mais surtout pendant que j'essayais de faire ça...

## 5. Replicate. 

Voici pour chunk2 : 
````
[mmederic@chunk2 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   20G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0   19G  0 part  
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
sdb           8:16   0    5G  0 disk  
├─sdb1        8:17   0    5G  0 part  
└─mpatha    253:2    0    5G  0 mpath 
  └─mpatha1 253:4    0    5G  0 part  /mnt/data_chunk1
sdc           8:32   0    5G  0 disk  
├─sdc1        8:33   0    5G  0 part  
└─mpathb    253:3    0    5G  0 mpath 
  └─mpathb1 253:5    0    5G  0 part  /mnt/data_chunk2
sdd           8:48   0    5G  0 disk  
├─sdd1        8:49   0    5G  0 part  
└─mpatha    253:2    0    5G  0 mpath 
  └─mpatha1 253:4    0    5G  0 part  /mnt/data_chunk1
sde           8:64   0    5G  0 disk  
├─sde1        8:65   0    5G  0 part  
└─mpathb    253:3    0    5G  0 mpath 
  └─mpathb1 253:5    0    5G  0 part  /mnt/data_chunk2
sr0          11:0    1 1024M  0 rom   
[mmederic@chunk2 ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                229M     0  229M   0% /dev/shm
tmpfs                 92M  3.4M   88M   4% /run
/dev/mapper/rl-root   17G  1.7G   16G  10% /
/dev/sda1            960M  431M  530M  45% /boot
tmpfs                 46M     0   46M   0% /run/user/1000
/dev/mapper/mpatha1  5.0G   68M  4.9G   2% /mnt/data_chunk1
/dev/mapper/mpathb1  5.0G   68M  4.9G   2% /mnt/data_chunk2
[mmederic@chunk2 ~]$ sudo iscsiadm -m session -P 3
iSCSI Transport Class version 2.0-870
version 6.2.1.9
Target: iqn.2024-12.tp2.b3:data-chunk2 (non-flash)
        Current Portal: 10.3.1.2:3260,1
        Persistent Portal: 10.3.1.2:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk2-initiator
                Iface IPaddress: 10.3.1.102
                Iface HWaddress: default
                Iface Netdev: default
                SID: 48
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 3  State: running
                scsi3 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdb          State: running
        Current Portal: 10.3.2.1:3260,1
        Persistent Portal: 10.3.2.1:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk2-initiator
                Iface IPaddress: 10.3.2.102
                Iface HWaddress: default
                Iface Netdev: default
                SID: 49
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 4  State: running
                scsi4 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sde          State: running
        Current Portal: 10.3.1.1:3260,1
        Persistent Portal: 10.3.1.1:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk2-initiator
                Iface IPaddress: 10.3.1.102
                Iface HWaddress: default
                Iface Netdev: default
                SID: 53
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 8  State: running
                scsi8 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdc          State: running
        Current Portal: 10.3.2.2:3260,1
        Persistent Portal: 10.3.2.2:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk2-initiator
                Iface IPaddress: 10.3.2.102
                Iface HWaddress: default
                Iface Netdev: default
                SID: 54
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 9  State: running
                scsi9 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdd          State: running
[mmederic@chunk2 ~]$ sudo multipath -ll
mpatha (36001405db562ce454f54628969266d41) dm-2 LIO-ORG,data-chunk2
size=5.0G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 3:0:0:0 sdb 8:16 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 9:0:0:0 sdd 8:48 active ready running
mpathb (360014053f73c389d13d4a11972fc461e) dm-3 LIO-ORG,data-chunk2
size=5.0G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 4:0:0:0 sde 8:64 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 8:0:0:0 sdc 8:32 active ready running

````

voici pour chunk3: 
````
[mmederic@chunk3 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda           8:0    0   20G  0 disk  
├─sda1        8:1    0    1G  0 part  /boot
└─sda2        8:2    0   19G  0 part  
  ├─rl-root 253:0    0   17G  0 lvm   /
  └─rl-swap 253:1    0    2G  0 lvm   [SWAP]
sdb           8:16   0    5G  0 disk  
└─mpatha    253:2    0    5G  0 mpath 
  └─mpatha1 253:4    0    5G  0 part  /mnt/data_chunk1
sdc           8:32   0    5G  0 disk  
└─mpathb    253:3    0    5G  0 mpath 
  └─mpathb1 253:5    0    5G  0 part  /mnt/data_chunk2
sdd           8:48   0    5G  0 disk  
└─mpathb    253:3    0    5G  0 mpath 
  └─mpathb1 253:5    0    5G  0 part  /mnt/data_chunk2
sde           8:64   0    5G  0 disk  
└─mpatha    253:2    0    5G  0 mpath 
  └─mpatha1 253:4    0    5G  0 part  /mnt/data_chunk1
sr0          11:0    1 1024M  0 rom   
[mmederic@chunk3 ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                229M     0  229M   0% /dev/shm
tmpfs                 92M  2.5M   89M   3% /run
/dev/mapper/rl-root   17G  1.7G   16G  10% /
/dev/sda1            960M  431M  530M  45% /boot
tmpfs                 46M     0   46M   0% /run/user/1000
/dev/mapper/mpatha1  5.0G   68M  4.9G   2% /mnt/data_chunk1
/dev/mapper/mpathb1  5.0G   68M  4.9G   2% /mnt/data_chunk2
[mmederic@chunk3 ~]$ sudo iscsiadm -m session -P 3
iSCSI Transport Class version 2.0-870
version 6.2.1.9
Target: iqn.2024-12.tp2.b3:data-chunk3 (non-flash)
        Current Portal: 10.3.1.1:3260,1
        Persistent Portal: 10.3.1.1:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk3:chunk3-initiator
                Iface IPaddress: 10.3.1.103
                Iface HWaddress: default
                Iface Netdev: default
                SID: 10
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 11 State: running
                scsi11 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdb          State: running
        Current Portal: 10.3.1.2:3260,1
        Persistent Portal: 10.3.1.2:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk3:chunk3-initiator
                Iface IPaddress: 10.3.1.103
                Iface HWaddress: default
                Iface Netdev: default
                SID: 11
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 12 State: running
                scsi12 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdc          State: running
        Current Portal: 10.3.2.1:3260,1
        Persistent Portal: 10.3.2.1:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk3:chunk3-initiator
                Iface IPaddress: 10.3.2.103
                Iface HWaddress: default
                Iface Netdev: default
                SID: 12
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 13 State: running
                scsi13 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sde          State: running
        Current Portal: 10.3.2.2:3260,1
        Persistent Portal: 10.3.2.2:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk3:chunk3-initiator
                Iface IPaddress: 10.3.2.103
                Iface HWaddress: default
                Iface Netdev: default
                SID: 3
                iSCSI Connection State: LOGGED IN
                iSCSI Session State: LOGGED_IN
                Internal iscsid Session State: NO CHANGE
                *********
                Timeouts:
                *********
                Recovery Timeout: 5
                Target Reset Timeout: 30
                LUN Reset Timeout: 30
                Abort Timeout: 15
                *****
                CHAP:
                *****
                username: <empty>
                password: ********
                username_in: <empty>
                password_in: ********
                ************************
                Negotiated iSCSI params:
                ************************
                HeaderDigest: None
                DataDigest: None
                MaxRecvDataSegmentLength: 262144
                MaxXmitDataSegmentLength: 262144
                FirstBurstLength: 65536
                MaxBurstLength: 262144
                ImmediateData: Yes
                InitialR2T: Yes
                MaxOutstandingR2T: 1
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 3  State: running
                scsi3 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sdd          State: running
[mmederic@chunk3 ~]$ sudo multipath -ll
mpatha (360014057fd41286d7d046b4a5b43e855) dm-2 LIO-ORG,data-chunk3
size=5.0G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 11:0:0:0 sdb 8:16 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 13:0:0:0 sde 8:64 active ready running
mpathb (3600140557bb1dc1ce1442fb807ead65d) dm-3 LIO-ORG,data-chunk3
size=5.0G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 12:0:0:0 sdc 8:32 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 3:0:0:0  sdd 8:48 active ready running
````

## III. Distributed Filesystem.

Jusqu'a présent on suit la doc de moosefs : 
````
[mmederic@master ~]$ sudo dnf install moosefs-master
[sudo] password for mmederic: 
MooseFS 9 - x86_64                                                                                                                                            17 kB/s | 6.1 kB     00:00    
Dependencies resolved.
=============================================================================================================================================================================================
 Package                                         Architecture                            Version                                              Repository                                Size
=============================================================================================================================================================================================
Installing:
 moosefs-master                                  x86_64                                  3.0.118-1.rhsystemd                                  MooseFS                                  379 k

Transaction Summary
=============================================================================================================================================================================================
Install  1 Package

Total download size: 379 k
Installed size: 873 k
Is this ok [y/N]: y
Downloading Packages:
moosefs-master-3.0.118-1.rhsystemd.x86_64.rpm                                                                                                                427 kB/s | 379 kB     00:00    
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                        424 kB/s | 379 kB     00:00     
MooseFS 9 - x86_64                                                                                                                                           1.7 MB/s | 1.8 kB     00:00    
Importing GPG key 0xCF82ADBA:
 Userid     : "MooseFS Development Team (Official MooseFS Repositories) <support@moosefs.com>"
 Fingerprint: 0F42 5A56 8FAF F43D EA2F 6843 0427 65FC CF82 ADBA
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-MooseFS
Is this ok [y/N]: y
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                     1/1 
  Running scriptlet: moosefs-master-3.0.118-1.rhsystemd.x86_64                                                                                                                           1/1 
  Installing       : moosefs-master-3.0.118-1.rhsystemd.x86_64                                                                                                                           1/1 
  Running scriptlet: moosefs-master-3.0.118-1.rhsystemd.x86_64                                                                                                                           1/1 
  Verifying        : moosefs-master-3.0.118-1.rhsystemd.x86_64                                                                                                                           1/1 

Installed:
  moosefs-master-3.0.118-1.rhsystemd.x86_64                                                                                                                                                  

Complete!
[mmederic@master ~]$ cd /etc/mfs
[mmederic@master mfs]$ cp mfsmaster.cfg.sample mfsmaster.cfg
cp: cannot create regular file 'mfsmaster.cfg': Permission denied
[mmederic@master mfs]$ sudo !!
sudo cp mfsmaster.cfg.sample mfsmaster.cfg
[mmederic@master mfs]$ ll
total 48
-rw-r--r--. 1 root root  6441 Dec  8 20:05 mfsexports.cfg
-rw-r--r--. 1 root root  6441 Aug  2 16:18 mfsexports.cfg.sample
-rw-r--r--. 1 root root 11618 Dec  8 20:05 mfsmaster.cfg
-rw-r--r--. 1 root root 11618 Aug  2 16:18 mfsmaster.cfg.sample
-rw-r--r--. 1 root root  2588 Dec  8 20:05 mfstopology.cfg
-rw-r--r--. 1 root root  2588 Aug  2 16:18 mfstopology.cfg.sample
[mmederic@master mfs]$ sudo cp mfsexports.cfg.sample mfsexports.cfg
````

On a du se root pour curl les liens moosefs.
````
[mmederic@master ~]$ su -
Password: 
Last login: Thu Dec  5 13:46:01 CET 2024 on tty1
[root@chunk1 ~]# curl "https://repository.moosefs.com/RPM-GPG-KEY-Moosefs" > /etc/pki/rpm-gpg/RPM-GPG-KEY-Moosefs
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   196  100   196    0     0    715      0 --:--:-- --:--:-- --:--:--   715
[root@master ~]# curl "https://repository.moosefs.com/MooseFS-3-el9.repo" > /etc/yum.repos.d/MooseFS.repo
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   175  100   175    0     0    660      0 --:--:-- --:--:-- --:--:--   660
```` 
Puis j'ai pris la doc de moosefs 3 pour les commandes de setup ci-dessus. 


On a ouvert nos chakr.. euh ports : 
````
[mmederic@master ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8 enp0s9
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 80/tcp 9425/tcp 9419/tcp 9420/tcp 9421/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
````

Ah oui, ça tourne au fait : 
````
[mmederic@master ~]$ sudo systemctl status moosefs-master
● moosefs-master.service - MooseFS Master server
     Loaded: loaded (/usr/lib/systemd/system/moosefs-master.service; enabled; preset: disabled)
     Active: active (running) since Sun 2024-12-08 20:11:11 CET; 3min 23s ago
    Process: 1487 ExecStart=/usr/sbin/mfsmaster start (code=exited, status=0/SUCCESS)
   Main PID: 1489 (mfsmaster)
      Tasks: 2 (limit: 2658)
     Memory: 125.3M
        CPU: 1.320s
     CGroup: /system.slice/moosefs-master.service
             ├─1489 /usr/sbin/mfsmaster start
             └─1490 "mfsmaster (data writer)"
````

#### 2. Chunk servers. 

Malheuresement je ne peux pas avancer plus car : 
````
Curl error (37): Couldn't read a file:// file for file:///etc/pki/rpm-gpg/RPM-GPG-KEY-MooseFS [Couldn't open file /etc/pki/rpm-gpg/RPM-GPG-KEY-MooseFS]
````

J'ai cherché pendant un moment sans rien trouvé. 

De plus allumer toutes les VMs (qui sont en 512 Mo et la GNS3Vm en 1024) 
fait péter mon Pc... La Ram sature. 
Par exemple je suis à 90% de pris sur ma ram avec master, chunk1, sto1 & sto2, et GNS3vm. 

Je suis sûre que ça fonctionne mais je suis limité actuellement.

Pas choqué mais déçu quand même, navré pour la fin de ce tp, que j'ai tout de meme lus (évidemment).

