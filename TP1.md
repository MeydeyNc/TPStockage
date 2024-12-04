# TP1 : Single Machine Storage. 

### I. Fundamentals 

Bonjour, bonsoir nous allons commencer par mettre en forme notre VM avec la petite checklist du plaisir. 
On a ssh tout le bazar avec la vm clonée. 

Nous avons commencé avec les disques : 
![disk](./assets/disks.jpg)

Puis le hostname : 
````
[mmederic@storage ~]$ hostname
storage.b3
````

On liste nos différents disques, avec uniquement le sda qui a ses partitions : 
````
[mmederic@storage ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk 
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   19G  0 part 
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0   10G  0 disk 
sdc           8:32   0   10G  0 disk 
sdd           8:48   0   10G  0 disk 
sde           8:64   0   10G  0 disk 
sdf           8:80   0   10G  0 disk 
sr0          11:0    1 1024M  0 rom  
````

On récupère entre temps smartmontools pour smartctl. 
Une petite recherche de docs sur internet, un search et on installe : 
````
[mmederic@storage ~]$ sudo dnf search smartmontools
[sudo] password for mmederic: 
Last metadata expiration check: 2:16:35 ago on Thu 14 Nov 2024 11:54:21 CET.
======================================= Name Exactly Matched: smartmontools ========================================
smartmontools.x86_64 : Tools for monitoring SMART capable hard disks
[mmederic@storage ~]$ sudo dnf install smartmontools -y
Installed:
  smartmontools-1:7.2-9.el9.x86_64                                                                 
Complete!
````

MAIS : 
````
[mmederic@storage ~]$ sudo smartctl -s on /dev/sda 
smartctl 7.2 2020-12-30 r5155 [x86_64-linux-5.14.0-427.42.1.el9_4.x86_64] (local build)
Copyright (C) 2002-20, Bruce Allen, Christian Franke, www.smartmontools.org

SMART support is: Unavailable - device lacks SMART capability.
A mandatory SMART command failed: exiting. To continue, add one or more '-T permissive' options.
````

On affiche l'espace disk disponible sur /, j'ai trouvé cette commande : 
````
[mmederic@storage ~]$ df -H /
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/rl-root   19G  1.7G   17G  10% /
````

Avec cette commande, on a la dispo des inodes restants et ceux utilisés, toujours sur / : 
````
[mmederic@storage ~]$ df -i / 
Filesystem           Inodes IUsed   IFree IUse% Mounted on
/dev/mapper/rl-root 8910848 37474 8873374    1% /
````

On install ioping via epel-release : 
````
[mmederic@storage ~]$ sudo dnf install epel-release -y
Complete !
[mmederic@storage ~]$ sudo dnf --enablerepo="epel" install ioping
Complete !
````

On fait notre premier test de latence sur /dev/sda : 
````
[mmederic@storage ~]$ sudo ioping -A /dev/sda
sudo ioping -A /dev/sda
4 KiB <<< /dev/sda (block device 20 GiB): request=1 time=7.26 ms (warmup)
4 KiB <<< /dev/sda (block device 20 GiB): request=2 time=3.98 ms
4 KiB <<< /dev/sda (block device 20 GiB): request=3 time=7.75 ms
4 KiB <<< /dev/sda (block device 20 GiB): request=4 time=618.8 us
4 KiB <<< /dev/sda (block device 20 GiB): request=5 time=725.1 us
^C
--- /dev/sda (block device 20 GiB) ioping statistics ---
4 requests completed in 13.1 ms, 16 KiB read, 306 iops, 1.20 MiB/s
generated 5 requests in 4.78 s, 20 KiB, 1 iops, 4.19 KiB/s
min/avg/max/mdev = 618.8 us / 3.27 ms / 7.75 ms / 2.92 ms
````

On a donc une latence moyenne de 3.27ms sur 5 ping. 

On va retrouver la taille de notre fscache : 
````
[mmederic@storage ~]$ free -h
               total        used        free      shared  buff/cache   available
Mem:           769Mi       191Mi       564Mi       0.0Ki       126Mi       577Mi
Swap:          2.0Gi        59Mi       1.9Gi
````

On a quelques manières différentes d'obtenir les infos de notre mémoire : 
````
[mmederic@storage ~]$ grep -E --color 'Mem|Cache|Swap|Buff' /proc/meminfo
MemTotal:         787688 kB
MemFree:          578080 kB
MemAvailable:     591512 kB
Buffers:               0 kB
Cached:           106760 kB
SwapCached:         4292 kB
SwapTotal:       2097148 kB
SwapFree:        2036056 kB
````

### II. Partitioning. 

On commence par check nos disk : 
````
[mmederic@storage ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk 
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   19G  0 part 
  ├─rl-root 253:0    0   17G  0 lvm  /
  └─rl-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0   10G  0 disk 
sdc           8:32   0   10G  0 disk 
sdd           8:48   0   10G  0 disk 
sde           8:64   0   10G  0 disk 
sdf           8:80   0   10G  0 disk 
sr0          11:0    1 1024M  0 rom
````

Puis on commence avec le PV de sdb : 
````
[mmederic@storage ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[mmederic@storage ~]$ sudo pvs
  PV         VG Fmt  Attr PSize   PFree 
  /dev/sda2  rl lvm2 a--  <19.00g     0 
  /dev/sdb      lvm2 ---   10.00g 10.00g
````

On suit avec le VG : 
````
[mmederic@storage ~]$ sudo vgcreate storage /dev/sdb
  Volume group "storage" successfully created
````

Puis on continue avec smol_data & big_data: 
````
[mmederic@storage ~]$ sudo lvcreate -L 2G storage -n smol_data
  Logical volume "smol_data" created.
[mmederic@storage ~]$ sudo lvcreate -L 7.99G storage -n big_data
  Rounding up size to full physical extent 7.99 GiB
  Logical volume "big_data" created.
[mmederic@storage ~]$ sudo lvs
  LV        VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root      rl      -wi-ao---- <17.00g                                                    
  swap      rl      -wi-ao----   2.00g                                                    
  big_data  storage -wi-a-----   7.99g                                                    
  smol_data storage -wi-a-----   2.00g   
````

On va créer le système de fichier en ext4 :
````
[mmederic@storage ~]$ sudo mkfs.ext4 /dev/storage/smol_data
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: 0a2532ab-a859-4d42-ab0f-97a9e8e27e3b
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

[mmederic@storage ~]$ sudo mkfs.ext4 /dev/storage/big_data
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 2095104 4k blocks and 524288 inodes
Filesystem UUID: 7de4c21e-9d95-40df-bab7-9dc13de10d99
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 
````

On va créer et monter les partitions : 
````
[mmederic@storage ~]$ sudo mkdir /mnt/lvm_storage/
[mmederic@storage ~]$ sudo mkdir /mnt/lvm_storage/smol
[mmederic@storage ~]$ sudo mount /dev/storage/smol_data /mnt/lvm_storage/smol
[mmederic@storage ~]$ sudo mkdir /mnt/lvm_storage/big
[mmederic@storage ~]$ sudo mount /dev/storage/big_data /mnt/lvm_storage/big
````

Les partitions sont bien montées : 
````
[mmederic@storage system]$ df -h
Filesystem                     Size  Used Avail Use% Mounted on
devtmpfs                       4.0M     0  4.0M   0% /dev
tmpfs                          385M     0  385M   0% /dev/shm
tmpfs                          154M  3.1M  151M   3% /run
/dev/mapper/rl-root             17G  1.8G   16G  11% /
/dev/sda1                      960M  309M  652M  33% /boot
tmpfs                           77M     0   77M   0% /run/user/1000
/dev/mapper/storage-smol_data  2.0G   24K  1.8G   1% /mnt/lvm_storage/smol
/dev/mapper/storage-big_data   7.8G   24K  7.4G   1% /mnt/lvm_storage/big
````

On umount pour le automount, on vérif et c'est bon : 
````
[mmederic@storage system]$ sudo umount /mnt/lvm_storage/smol
[mmederic@storage system]$ sudo umount /mnt/lvm_storage/big
[mmederic@storage system]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                385M     0  385M   0% /dev/shm
tmpfs                154M  3.1M  151M   3% /run
/dev/mapper/rl-root   17G  1.8G   16G  11% /
/dev/sda1            960M  309M  652M  33% /boot
tmpfs                 77M     0   77M   0% /run/user/1000
````

### III. RAID. 

#### 1. Simple RAID.

On update, puis on récupère mdadm. 
````
sudo dnf update -y
sudo dnf search mdadm 
sudo dnf install mdadm -y
````

Avec le internet nous sachons le RAID5 : 
````
[mmederic@storage ~]$ sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdc /dev/sdd /dev/sde 
[sudo] password for mmederic: 
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 10476544K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
````
Avec une tite commande on peut voir notre raid : 
````
[mmederic@storage ~]$ lsblk | grep ['sdc','sdd','sde'] | tail -n 8
sdc                   8:32   0   10G  0 disk  
└─md0                 9:0    0   20G  0 raid5 /mnt/raid5
sdd                   8:48   0   10G  0 disk  
└─md0                 9:0    0   20G  0 raid5 /mnt/raid5
sde                   8:64   0   10G  0 disk  
└─md0                 9:0    0   20G  0 raid5 /mnt/raid5
sdf                   8:80   0   10G  0 disk  
sr0                  11:0    1 1024M  0 rom   
````
sdc, sdd & sde sont en md0. C'est un bon signe ça nan ? 

On refait une tite commande parce que c'est marrant (et on espère que c'est plus lisible) :
````
[mmederic@storage ~]$ lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT | grep ['sdc','sdd','sde'] | tail -n 8
sdc                   10G linux_raid_member disk  
└─md0                 20G ext4              raid5 /mnt/raid5
sdd                   10G linux_raid_member disk  
└─md0                 20G ext4              raid5 /mnt/raid5
sde                   10G linux_raid_member disk  
└─md0                 20G ext4              raid5 /mnt/raid5
sdf                   10G                   disk  
sr0                 1024M                   rom  
````

Bon, j'avoue j'abuse, juste je connaissais pas cette commande : 
````
[mmederic@storage ~]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:44:47 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 09:50:00 2024
             State : clean 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 47e195e2:62aea9ad:6393b86e:f98df2fe
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde
````

Pour faire en sorte d'automatiser la configuration au démarrage on est allés trouver une commande un peu obscure sur [ubuntu](https://doc.ubuntu-fr.org/raid_logiciel) : 
````
[mmederic@storage ~]$ sudo mdadm --monitor --daemonise /dev/md0
4465
````

Voici notre conf mdadm : 
````
[mmederic@storage ~]$ cat /etc/mdadm/mdadm.conf 
# mdadm.conf
#
# Please refer to mdadm.conf(5) for information about this file.
#

DEVICE partitions

# auto-create devices with Debian standard permissions
CREATE owner=root group=disk mode=0660 auto=yes

# automatically tag new arrays as belonging to the local system
HOMEHOST <system>

# instruct the monitoring daemon where to send mail alerts
MAILADDR root

# definitions of existing MD arrays
ARRAY /dev/md0 metadata=1.2 UUID=47e195e2:62aea9ad:6393b86e:f98df2fe
````

On a use un peu de ext4 : 
````
[mmederic@storage ~]$ sudo mkfs.ext4 /dev/md0 
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 5238272 4k blocks and 1310720 inodes
Filesystem UUID: 6e9ed206-3aed-4922-ace1-0995499a59db
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done 
````

On monte la partition : 
````
[mmederic@storage ~]$ sudo mount /dev/md0 /mnt/raid_storage
[mmederic@storage ~]$ lsblk | grep 'raid_storage'
└─md0                 9:0    0   20G  0 raid5 /mnt/raid_storage
└─md0                 9:0    0   20G  0 raid5 /mnt/raid_storage
└─md0                 9:0    0   20G  0 raid5 /mnt/raid_storage
````

Ca écrit et ça lit : 
````
[mmederic@storage ~]$ cat /mnt/raid_storage/toto.txt 
oui
````

#### 2. Break it. 

On vient de débrancher un disk du raid, situtation dégradé : 

Avec la commande c'est mieux : 
````
sudo mdadm --manage /dev/md0 --set-faulty /dev/sdd
````

````
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0 
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:44:47 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 16:58:28 2024
             State : clean, degraded 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 47e195e2:62aea9ad:6393b86e:f98df2fe
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       -       0        0        1      removed
       3       8       48        2      active sync   /dev/sdd
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0 | grep degraded
             State : clean, degraded 
````

Tout s'est remis en ordre automatiquement : 
````
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0 
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:44:47 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 16:58:28 2024
             State : clean 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 47e195e2:62aea9ad:6393b86e:f98df2fe
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde
````

Tout semble fonctionnel : 
````
[mmederic@storage ~]$ cat /proc/mdstat 
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sde[3] sdd[1] sdc[0]
      20953088 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      
unused devices: <none>
````

#### 3. Spare disk

On va ajouter un disk en spare & on vérifie : 
````
[mmederic@storage ~]$ sudo mdadm --manage /dev/md0 --add /dev/sdf 
mdadm: added /dev/sdf
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0 
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:44:47 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 17:32:48 2024
             State : clean 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 47e195e2:62aea9ad:6393b86e:f98df2fe
            Events : 21

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde

       4       8       80        -      spare   /dev/sdf
````

On a bien sdf en spare. 

On va alors simuler une panne again. 
````
[mmederic@storage ~]$ sudo mdadm --manage /dev/md0 --set-faulty /dev/sdd 
mdadm: set /dev/sdd faulty in /dev/md0
````

Puis on observe :
````
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:44:47 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 17:38:03 2024
             State : clean, degraded, recovering 
    Active Devices : 2
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 24% complete

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 47e195e2:62aea9ad:6393b86e:f98df2fe
            Events : 26

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       4       8       80        1      spare rebuilding   /dev/sdf
       3       8       64        2      active sync   /dev/sde

       1       8       48        -      faulty   /dev/sdd
````
Ce qui est cool c'est qu'on voit bien le rebuild du spare avec le faulty. 
On en fait plusieurs de cette commande et on a le compte qui augmente au fur et à mesure du rebuild.

Et c'est bon : 
````
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:44:47 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 17:38:46 2024
             State : clean 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 47e195e2:62aea9ad:6393b86e:f98df2fe
            Events : 40

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       4       8       80        1      active sync   /dev/sdf
       3       8       64        2      active sync   /dev/sde

       1       8       48        -      faulty   /dev/sdd

````

On vient de remettre sdd : 
````
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:44:47 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Dec  4 17:41:30 2024
             State : clean 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : 47e195e2:62aea9ad:6393b86e:f98df2fe
            Events : 42

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       4       8       80        1      active sync   /dev/sdf
       3       8       64        2      active sync   /dev/sde

       5       8       48        -      spare   /dev/sdd

````

On peut voir que c'est sdd qui est passé en spare maintenant. 
SDF a pris la place de sdd dans le RAID pour le fonctionnel. 
Interesting. 

#### 4. Grow

On va add un noveau spare & on vérifie: 
````
[mmederic@storage ~]$ sudo mdadm --manage /dev/md0 --add /dev/sdg  
mdadm: added /dev/sdg
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0 | grep Spare
     Spare Devices : 2
````

La bible du mdadm trouvée plus tot fais des merveilles, petite commande simple pour agrandir le raid :
````
[mmederic@storage ~]$ sudo mdadm --grow /dev/md0 --raid-devices=4
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0 | grep 'Active Devices'
    Active Devices : 4
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0 | grep 'Spare Devices'
     Spare Devices : 1
````

Tout s'est bien passé. 

Pour le coup, la taille est tjrs à 30G : 
````
└─md0                 9:0    0   30G  0 raid5
````

J'ai fais mes commandes, mais rien y change : 
````
[mmederic@storage ~]$ sudo e2fsck -f -y -v -C 0 '/dev/md0' 
e2fsck 1.46.5 (30-Dec-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure                                           
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information                                     
                                                                               
          11 inodes used (0.00%, out of 1966080)
           0 non-contiguous files (0.0%)
           0 non-contiguous directories (0.0%)
             # of inodes with ind/dind/tind blocks: 0/0/0
             Extent depth histogram: 3
      167453 blocks used (2.13%, out of 7858560)
           0 bad blocks
           1 large file

           0 regular files
           2 directories
           0 character device files
           0 block device files
           0 fifos
           0 links
           0 symbolic links (0 fast symbolic links)
           0 sockets
------------
           2 files
[mmederic@storage ~]$ sudo resize2fs /dev/md0 
resize2fs 1.46.5 (30-Dec-2021)
The filesystem is already 7858560 (4k) blocks long.  Nothing to do!
[mmederic@storage ~]$ sudo mdadm --detail /dev/md0 
[sudo] password for mmederic: 
/dev/md0:
           Version : 1.2
     Creation Time : Wed Dec  4 09:44:47 2024
        Raid Level : raid5
        Array Size : 31434240 (29.98 GiB 32.19 GB)
     Used Dev Size : 10478080 (9.99 GiB 10.73 GB)

````

Je reste bloqué à 30G. Pas 40. Idk :v osecour

### IV. NFS

On installe NFS : 
````
sudo dnf install nfs-utils -y
````

Après quelques corrections dans mon lvm_storage.mount, le nfs-server fonctionne :
````
[mmederic@storage ~]$ sudo systemctl status nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: disabled)
    Drop-In: /run/systemd/generator/nfs-server.service.d
             └─order-with-mounts.conf
     Active: active (exited) since Wed 2024-12-04 19:05:22 CET; 43s ago
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
    Process: 2319 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 2320 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
    Process: 2336 ExecStart=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=>
   Main PID: 2336 (code=exited, status=0/SUCCESS)
        CPU: 43ms

Dec 04 19:05:22 storage.b3 systemd[1]: Starting NFS server and services...
Dec 04 19:05:22 storage.b3 systemd[1]: Finished NFS server and services.
lines 1-15/15 (END)
````

la correction en question : 
````
[mmederic@storage ~]$ cat /etc/systemd/system/lvm_storage.mount 
[Mount]
What=/dev/sdb
Where=/mnt/lvm_storage/smol/
Type=ext4
````

On crée les chemins de fichiers, on les monte : 
````
[mmederic@tp1 ~]$ sudo mkdir -p /mnt/lvm_storage
[sudo] password for mmederic: 
[mmederic@tp1 ~]$ sudo mkdir -p /mnt/raid_storage
[mmederic@tp1 ~]$ sudo mount 10.10.0.10:/mnt/lvm_storage /mnt/lvm_storage/
[mmederic@tp1 ~]$ sudo mount 10.10.0.10:/mnt/raid_storage/ /mnt/raid_storage/
[mmederic@tp1 ~]$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
devtmpfs                      4.0M     0  4.0M   0% /dev
tmpfs                         385M     0  385M   0% /dev/shm
tmpfs                         154M  3.1M  151M   2% /run
/dev/mapper/rl-root            17G  1.7G   16G  10% /
/dev/sda1                     960M  401M  560M  42% /boot
tmpfs                          77M     0   77M   0% /run/user/1000
10.10.0.10:/mnt/lvm_storage    17G  1.9G   16G  11% /mnt/lvm_storage
10.10.0.10:/mnt/raid_storage   30G     0   28G   0% /mnt/raid_storage
[mmederic@tp1 ~]$ 
````
Et paf ! ça fait des chocapics. 

On test la création du truk la : 
````
[mmederic@tp1 ~]$ sudo touch /mnt/lvm_storage/smol/toto.txt
[mmederic@tp1 ~]$ ls  /mnt/lvm_storage/smol/
toto.txt
````

le test, lvm : 
````
[mmederic@tp1 ~]$ dd if=/mnt/lvm_storage/smol/toto.txt of=/tmp/output
0+0 records in
0+0 records out
0 bytes copied, 0.00289999 s, 0.0 kB/s
````

le test, raid :
````
[mmederic@tp1 ~]$ dd if=/mnt/raid_storage/toto.txt of=/tmp/output
0+0 records in
0+0 records out
0 bytes copied, 0.00116726 s, 0.0 kB/s
````

Bon le test est nul, j'avoue je sais pas faire les vré tests tout ça, mais on voit que ça va plus vite avec le raid envré. 

En tout cas, c'était sympa, mais je vais surement avoir besoin de la doc à nouveau pour le refaire un jour !