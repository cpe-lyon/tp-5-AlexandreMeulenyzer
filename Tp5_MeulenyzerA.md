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
```

### Question 2 : . Supprimez les deux partitions du disque, et créez une patition unique de type LVM


LVM :


* appuyer sur n pour créé une nouvelle partition

* appuyer sur p pour créer une partition primaire (4max)

* appuyer sur 1 pour denote it as 1st disk partition

* appuyer sur t pour changer le type de partition par défaut Linux (0x83) en LVM partition (0x8e),

* appuyer sur L pour lister toutes les types de partitions

* appuyer sur 8e pour changer la partition de 1 à 8e

* appuyer sur p pour afficher le groupe du 2ème disque. 
  
* appuyer sur w pour écrire les partitions dans la table.

### Question 3 : A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay.

pvcreate :
```bash

[10:11]-[root]@client-/home/User: pvcreate /dev/sdb1

  Physical volume "/dev/sdb1" successfully created.

[10:12]-[root]@client-/home/User: pvdisplay

  "/dev/sdb1" is a new physical volume of "<5,00 GiB"

  --- NEW Physical volume ---

  PV Name               /dev/sdb1

  VG Name

  PV Size               <5,00 GiB

  Allocatable           NO

  PE Size               0

  Total PE              0

  Free PE               0

  Allocated PE          0

  PV UUID               V47ZaY-LQ8N-t1MI-zK9H-tUvZ-Gn1m-ElNYiM
```
### Question 4 : A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente Vérifiez à l’aide de la commande vgdisplay. 


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

### Question 9 : Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.

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
