# Exercice 1. Disques et partitions

### Question 1 : Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM
  
  Disque créé

### Question 2 : Vérifiez que ce nouveau disque dur est bien détecté par le système

commande : fdisk -l

Vérification :
```bash
1
Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors
2
Disk model: Virtual disk
3
Units: sectors of 1 * 512 = 512 bytes
4
Sector size (logical/physical): 512 bytes / 512 bytes
5
I/O size (minimum/optimal): 512 bytes / 512 bytes
Question 3 : Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)
```

Pour faire ceci, il faut utiliser la commande : fdisk /dev/"nom du disque" par exemple fdisk /dev/sdb, puis suivre les commande affiché
Partitions en NTFS :

```bash
1
fdisk /dev/sdb
2
Commands:
3
​
4
to create the partition: n, p, [enter], [enter]
5
to give a type to the partition: t, 7 (dont select 86 or 87, those are for volume sets)
6
if you want to make it bootable: a
7
to see the changes: p
8
to write the changes: w
9
mkfs.ntfs -f /dev/sdb2
10
mkdir /mnt/ntfsvolume
11
mount /dev/sdb2  /mnt/ntfsvolume
```

### Question 4 : A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions (! pensez à consulter le manuel !)

La partition en NTFS a été formatée à la question précédente, pour ce qui est de celle qui est de type Linux il faut la faire de la manière suivante :
mkfs.ext4 /dev/sdb1

### Question 5 : Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?

La commande df -T ne fonctionne pas car nos partitions ne sont pas montées. Il faut absolument avoir un système de fichiers et que la partition / disque soit monté pour que cela fonctionne.

### Question 6 : Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)

Les commandes pour que les deux partitions soient montés sont :

```bash
1
mkdir /data
2
[09:52]-[root]@client-/home/User: mkdir /win
3
[09:52]-[root]@client-/home/User: umount /dev/sdb2
4
[09:52]-[root]@client-/home/User: mount /dev/sdb2 /win
5
[09:52]-[root]@client-/home/User: mount /dev/sdb1 /data
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
```

### Question 2 : . Supprimez les deux partitions du disque, et créez une patition unique de type LVM
Astuce : La création d’une partition LVM n’est pas indispensable, mais vivement recommandée quand on utilise LVM sur un disque entier. En effet, elle permet d’indiquer à d’autres OS ou logiciels de gestion de disques (qui ne reconnaissent pas forcément le format LVM) qu’il y a des données sur ce disque.
Attention à ne pas supprimer la partition système !

LVM :


* appuyer sur n pour créé une nouvelle partition

* appuyer sur p pour créer une partition primaire (4max)

* appuyer sur 1 pour denote it as 1st disk partition

* appuyer sur t pour changer le type de partition par défaut Linux (0x83) en LVM partition (0x8e),

* appuyer sur L pour lister toutes les types de partitions

* appuyer sur 8e pour changer la partition de 1 à 8e

* appuyer sur p pour afficher le groupe du 2ème disque. 
  
* appuyer sur w pour écrire les partitions dans la table.

### Question 3 : A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay. Toutes les commandes concernant les volumes physiques commencent par pv. Celles concernant les groupes de volumes commencent par vg, et celles concernant les volumes logiques par lv.

pvcreate :
```bash
1
[10:11]-[root]@client-/home/User: pvcreate /dev/sdb1
2
  Physical volume "/dev/sdb1" successfully created.
3
[10:12]-[root]@client-/home/User: pvdisplay
4
  "/dev/sdb1" is a new physical volume of "<5,00 GiB"
5
  --- NEW Physical volume ---
6
  PV Name               /dev/sdb1
7
  VG Name
8
  PV Size               <5,00 GiB
9
  Allocatable           NO
10
  PE Size               0
11
  Total PE              0
12
  Free PE               0
13
  Allocated PE          0
14
  PV UUID               V47ZaY-LQ8N-t1MI-zK9H-tUvZ-Gn1m-ElNYiM
```
### Question 4 : A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente Vérifiez à l’aide de la commande vgdisplay. 
### Par convention, on nomme généralement les groupes de volumes vgxx (où xx représente l’indice du groupe de volume, en commençant par 00, puis 01...)

 vgcreateetvgdisplay :
```bash

[10:12]-[root]@client-/home/User: vgcreate vg00 /dev/sdb1

  Volume group "vg00" successfully created

[10:14]-[root]@client-/home/User: vgdisplay

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

  VG Size               <5,00 GiB

  PE Size               4,00 MiB
  Total PE              1279

  Alloc PE / Size       0 / 0

  Free  PE / Size       1279 / <5,00 GiB

  VG UUID               IRsQ73-4dTc-4aaM-h0Fm-NYfh-n0My-DocV1t

```
### Question 5 : Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.
On peut renseigner la taille d’un volume logique soit de manière absolue avec l’option -L (par exemple -L 10G pour créer un volume de 10 Gio), soit de manière relative avec l’option -l : -l 60%VG pour utiliser 60% de l’espace total du groupe de volumes, ou encore -l 100%FREE pour utiliser la totalité de l’espace libre.

Création du volume :
```bash

lvcreate -l 100%FREE vg00

Logical volume "lvol0" created.

lvrename vg00 lvol0 lvdata

Renamed "lvol0" to "lvdata" in volume group "vg00"
```


### Question 7 : Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque

Le disque ajouté est bien visible, donc nous procédons suppression des deux partitions du disque, et la création une patition unique de type LVM, puis du formatage LVM

### Question 8 : Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes

Voici la commande entrée :
```bash
vgextend vg00 /dev/sdc
```

### Question 9 : Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs. Il est possible d’aller beaucoup plus loin avec LVM, par exemple en créant des volumes par bandes (l’équivalent du RAID 0) ou du mirroring (RAID 1). Le but de cet exercice n’était que de présenter les fonctionnalités de base

Il faut entrer les commandes dans l'ordre suivant :
```bash
lvextend -L +2.9G /dev/mapper/vg00-lvdata
resize2fs  /dev/mapper/vg00-lvdata
```
```bash
[10:43]-[root]@client-/home/User: lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg00/lvdata
  LV Name                lvdata
  VG Name                vg00
  LV UUID                yiyQbF-ju9q-YraK-xtnj-osF0-cIp0-4u45i4
  LV Write Access        read/write
  LV Status              available
  # open                 1
  LV Size                <7,90 GiB
  Current LE             2022
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

lvextend -L +2.9G /dev/mapper/vg00-lvdata

resize2fs  /dev/mapper/vg00-lvdata

[10:43]-[root]@client-/home/User: lvdisplay

  --- Logical volume ---

  LV Path                /dev/vg00/lvdata

  LV Name                lvdata

  VG Name                vg00

  LV UUID                yiyQbF-ju9q-YraK-xtnj-osF0-cIp0-4u45i4

  LV Write Access        read/write

  LV Status              available

  # open                 1

  LV Size                <7,90 GiB

  Current LE             2022

  Segments               2

  Allocation             inherit

  Read ahead sectors     auto

  - currently set to     256

  Block device           253:0
```