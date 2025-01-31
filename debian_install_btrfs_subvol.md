# Установка Debian 12 на подтома (subvolumes) BTRFS  
В отличии от инсталлятора Ubuntu, в инсталляторе Debian нет возможности 
настроить монтирование стандартных каталогов, таких как `/`, `/home`, `/var` и
подобных на подтома BTRFS в одном разделе. Инсталлятор Debian создаёт 
только 1 подтом `@rootfs` для монтирования корня.
Для того чтобы настроить монтирование стандартных каталогов в подтома BTRFS, 
надо во время инсталляци произвести определённые манипуляции из консоли.  
В меню GRUB надо выбрать `Advanced options...`, там выбрать `expert install...`.  
Установку производим как обычно.  
Для удобства в пункте
`Загрузка компонентов с установочного носителя` можно выбрать модули
`choose-mirror` и `network-install`. Модуль `choose-mirror` позволяет выбрать зеркало
Debian в Интернете, модуль `network-install` позволяет производить инсталляцию с
другого компьютера, подключившись по ssh. Это позволяет редактировать содержимого файла `fstab`
в удобном редакторе.  
Диск размечаем вручную по своему усмотрению, создаём раздел для `/` c файловой системой BTRFS.  
Например, такая разметка:
```
dev/nvme0n1 - 2.0 TB Samsung SSD 990 PRO with Heatsink 2TB
>             1.0 MB        СВОБОДНОЕ МЕСТО
>     #1    267.4 MB  B  K  ESP
>     #2     34.4 GB     F  подк               подк
>     #3      2.0 TB     F  btrfs              /
>            73.2 kB        СВОБОДНОЕ МЕСТО
```
После завершения разметки выбираем в меню пункт 
`Закончить разметку и записать изменения на диск`.  
Теперь запускаем консоль. При локальной инсталляции нажимаем `Ctrl`+`Alt`+`F2`.
При сетевой инсталляции выбрать в меню инсталляции пункт `Запуск оболочки`.
Размонтируем разделы, которые смонтировал инсталлятор на диске, куда будем ставить систему:
```bash
umount /target/boot/efi
umount /target
```
Монтируем созданный инсталлятором подтом `@rootfs` в другое место. Оттуда надо будет взять файл
`/etc/fstab`, после этого уничтожим её.
```bash
mkdir /mnt/rootfs
mount /dev/nvme0n1p3 -o subvol=@rootfs,discard,defaults /mnt/rootfs
```
Монтируем сам раздел BTRFS, в котором будем создавать подтома для файловых систем.
```bash
mkdir /mnt/new
mount -o defaults,discard /dev/nvme0n1p3 /mnt/new
```
Создаём подтома в BTRFS:
```bash
cd /mnt/new
btrfs subvolume create @
btrfs subvolume create @var
btrfs subvolume create @home
btrfs subvolume create @.snapshots
```
Проверим список томов:
```bash
btrfs subvolume list .
```
```
ID 256 gen 34 top level 5 path @rootfs
ID 261 gen 36 top level 5 path @
ID 262 gen 37 top level 5 path @var
ID 263 gen 38 top level 5 path @home
ID 264 gen 39 top level 5 path @.snapshots
```
Теперь монтируем подтома, куда будет инсталлироваться система:
```shell
mount -o subvol=@,defaults,discard /dev/nvme0n1p3 /target

mkdir -p /target/boot/efi
mkdir -p /target/home
mkdir -p /target/var
mkdir -p /target/.snapshots

mount /dev/nvme0n1p3 -o defaults,discard,subvol=@home /target/home
mount /dev/nvme0n1p3 -o defaults,discard,subvol=@var /target/var
mount /dev/nvme0n1p3 -o defaults,discard,subvol=@.snapshots /target/.snapshots
mount /dev/nvme0n1p1 /target/boot/efi
```
Теперь переходим к редактированию файла `/target/etc/fstab`:
```shell
cat /mnt/rootfs/etc/fstab
```
```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# systemd generates mount units based on this file, see systemd.mount(5).
# Please run 'systemctl daemon-reload' after making changes here.
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/nvme0n1p3 during installation
UUID=05c5c076-6bd1-470d-b14f-9c95920bbe11 /               btrfs   defaults,subvol=@rootfs 0       0
# /boot/efi was on /dev/nvme0n1p1 during installation
UUID=77F3-C125  /boot/efi       vfat    umask=0077      0       1
# swap was on /dev/nvme0n1p2 during installation
UUID=8504686c-eadb-4d0e-9aad-a0ece664a2ab none            swap    sw              0       0
```
В файле `/target/etc/fstab` прописываем подтома в нашем разделе BTRFS.
```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# systemd generates mount units based on this file, see systemd.mount(5).
# Please run 'systemctl daemon-reload' after making changes here.
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/nvme0n1p3 during installation
UUID=05c5c076-6bd1-470d-b14f-9c95920bbe11 /               btrfs   defaults,subvol=@,discard 0       0
UUID=05c5c076-6bd1-470d-b14f-9c95920bbe11 /home               btrfs   defaults,subvol=@home,discard 0       0
UUID=05c5c076-6bd1-470d-b14f-9c95920bbe11 /var               btrfs   defaults,subvol=@var,discard 0       0
UUID=05c5c076-6bd1-470d-b14f-9c95920bbe11 /.snapshots               btrfs   defaults,subvol=@.snapshots,discard 0       0
# /boot/efi was on /dev/nvme0n1p1 during installation
UUID=77F3-C125  /boot/efi       vfat    umask=0077      0       1
# swap was on /dev/nvme0n1p2 during installation
UUID=8504686c-eadb-4d0e-9aad-a0ece664a2ab none            swap    sw              0       0
```
Теперь подтом `@rootfs`, созданный установщиком Debian, можно уничтожить.
```shell
cd /
umount /mnt/rootfs
cd /mnt/new
btrfs subvolume delete @rootfs
cd /
umount /mnt/new
```
Имена подтомов выбраны такими для совместимости с программами `timeshift` и `snapper`. 
Docker также можно настроить чтобы он использовал для образов BTRFS, а не OverlayFS. 
Теперь выходим из терминала. При локальной установке возвращаемся к установщику по комбинации 
`Ctrl`+`Alt`+`F1`, а при сетевой выходим из терминала командой `exit`.   
Продолжаем установку: в меню выбираем следующий пункт: `установка базовой системы`.

