# Exercice 1. Disques et partitions

### Question 1 : Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM
  
  Ajout de disque : démarrage de la vm, disque bien pris en compte.

### Question 2 : Vérifiez que ce nouveau disque dur est bien détecté par le système

commande : fdisk -l

Vérification :
```bash

Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors

Disk model: Virtual disk

Units: sectors of 1 * 512 = 512 bytes

Sector size (logical/physical): 512 bytes / 512 bytes

I/O size (minimum/optimal): 512 bytes / 512 bytes

```
### Question 3 : Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)
fdisk /dev/sdb, puis suivre les commande affiché
Partitions en NTFS :

```bash

fdisk /dev/sdb

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-10485759, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (4196352-10485759, default 4196352): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-10485759, default 10485759): 

Created a new partition 2 of type 'Linux' and of size 3 GiB.

```

### Question 4 : A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions (! pensez à consulter le manuel !)

mkfs.ext4 /dev/sdb1
mfks.ntfs /dev/sdb2

### Question 5 : Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?

La commande df -T ne fonctionne pas car nos partitions ne sont pas montées.

### Question 6 : Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)


```bash

root@serveur:/home/User# mount /dev/sdb2 /win/
root@serveur:/home/User# mount /dev/sdb1 /data/
```
Pour que les fichiers soient montés automatiquement au démarrage : il faut modifier le fichier /etc/fstab

### Question 7 : Utilisez la commande mount puis redémarrez votre VM pour valider la configuration
  
df -T :
| Filesystem | Type     | 1K-blocks | Used    | Available              | Use% | Mounted on        |
|------------|----------|-----------|---------|------------------------|------|-------------------|
| udev       | devtmpfs | 202128    | 0       | 202128                 | 0%   | /dev              |
| tmpfs      | tmpfs    | 48852     | 1116    | 47736                  | 3%   | /run              |
| /dev/sda2  | ext4     | 16445308  | 5953616 | 9636604                | 39%  | /                 |
| tmpfs      | tmpfs    | 244244    | 0       | 244244                 | 0%   | /dev/shm          |
| tmpfs      | tmpfs    | 5120      | 0       | 5120                   | 0%   | /run/lock         |
| tmpfs      | tmpfs    | 244244    | 0       | 244244                 | 0%   | /sys/fs/cgroup    |
| /dev/loop0 | squashfs | 96256     | 96256   | 0 100%                 | 100% | /snap/core/9106   |
| /dev/loop1 | squashfs | 69376     | 69376   | 0 100%                 | 100% | /snap/lxd/15105   |
| /dev/loop2 | squashfs | 56320     | 56320   | 0 100%                 | 100% | /snap/core18/1864 |
| /dev/loop3 | squashfs | 70912     | 70912   | 0 100% /snap/lxd/14954 | 100% | /snap/lxd/14458   |
| /dev/loop4 | squashfs | 96128     | 96128   | 0 100% /snap/core/8935 | 100% | /snap/core/9254   |
| /dev/sdb1  | ext4     | 1998672   | 6144    | 1871288                | 1%   | /data             |
| /dev/sdb2  | fuseblk  | 3144700   | 16264   | 3128436                | 1%   | /win              |
| tmpfs      | tmpfs    | 48848     | 0       | 48848                  | 0%   | /run/user/1000    |
### Question 8 et 9 passé

# Exercice 2. Partitionnement LVM
Dans cet exercice, nous allons aborder le partitionnement LVM, beaucoup plus flexible pour manipuler les disques et les partitions.

### Question 1 : On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes du fichier /etc/fstab

```bash
umount /dev/sdb1
umount /dev/sdb2
```

### Question 2 : . Supprimez les deux partitions du disque, et créez une patition unique de type LVM


LVM :
```bash
Command (m for help): t
Selected partition 1
Hex code or alias (type L to list all): 8e
Changed type of partition 'Linux' to 'Linux LVM'.
```
### Question 3 : A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay.

pvcreate :
```bash

root@serveur:/home/User# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
root@serveur:/home/User# pvdisplay 
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               sysvg
  PV Size               <38.00 GiB / not usable 2.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              9727
  Free PE               1279
  Allocated PE          8448
  PV UUID               wQIWMI-3vhz-xBTe-TItJ-wtvS-AE3I-3wZGd4
   
  "/dev/sdb1" is a new physical volume of "<5.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name               
  PV Size               <5.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               QLb4g5-QZHx-UYGQ-pTgL-xV4O-t8n5-ickbXt
```
### Question 4 : A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente Vérifiez à l’aide de la commande vgdisplay. 

```bash

root@serveur:/home/User# vgcreate vg00 /dev/sdb1
  Volume group "vg00" successfully created
root@serveur:/home/User# vgdisplay
  --- Volume group ---
  VG Name               vg00
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5.00 GiB
  PE Size               4.00 MiB
  Total PE              1279
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1279 / <5.00 GiB
  VG UUID               34iXSm-QCl4-Ah5n-ngP3-yAt1-9BLw-i9g2nG
   
  --- Volume group ---
  VG Name               sysvg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  8
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                7
  Open LV               7
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <38.00 GiB
  PE Size               4.00 MiB
  Total PE              9727
  Alloc PE / Size       8448 / 33.00 GiB
  Free  PE / Size       1279 / <5.00 GiB
  VG UUID               UC41XH-YKj4-8g4u-MZkc-a1we-7W27-g9xMRV

```
### Question 5 : Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.


Création du volume :
```bash

root@serveur:/home/User# lvcreate -l 100%FREE vg00
  Logical volume "lvol0" created.
root@serveur:/home/User# lvrename vg00 lvol0 lvdata
  Renamed "lvol0" to "lvdata" in volume group "vg00"
```

### Question 6 :
```bash
root@serveur:/home/User# fdisk /dev/vg00/lvdata 

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-10477567, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10477567, default 10477567): 

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help): t
Selected partition 1
Hex code or alias (type L to list all): 83
Changed type of partition 'Empty' to 'Linux'.
```
### Question 7 : Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque

```bash
root@serveur:/home/User# fdisk /dev/sdc 

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x01d3930d.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-4194303, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-4194303, default 4194303): 

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): 

Selected partition 1
Hex code or alias (type L to list all): 8e
Changed type of partition 'Linux' to 'Linux LVM'.
```
```bash
root@serveur:/home/User# pvdisplay /dev/sdc1
  "/dev/sdc1" is a new physical volume of "<2.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc1
  VG Name               
  PV Size               <2.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               rmViL7-QcJX-xRK7-4iQk-Ak3w-Jtst-slH3mV
  ```
### Question 8 : Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes

Voici la commande entrée :
```bash
root@serveur:/home/User# vgextend vg00 /dev/sdc1
  Volume group "vg00" successfully extended
```

### Question 9 : Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.

```bash
root@serveur:/home/User# lvextend -L +1.5G /dev/mapper/vg00-lvdata 
  Size of logical volume vg00/lvdata changed from <5.00 GiB (1279 extents) to <6.50 GiB (1663 extents).
  Logical volume vg00/lvdata successfully resized.
root@serveur:/home/User# resize
resize2fs   resizecons  resizepart  
root@serveur:/home/User# resize2fs /dev/mapper/vg00-lvdata 
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/mapper/vg00-lvdata is mounted on /data; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/vg00-lvdata is now 1702912 (4k) blocks long.

root@serveur:/home/User# 
```
