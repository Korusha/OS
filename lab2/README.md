# Описание лабораторной работы №2
## Задание 1. Установка ОС и настройка LVM, RAID
#### 1. Начало работы. Установила Virtual Box, затем начала установку Linux, где дошла до выбора жестких дисков. 
Выбрала Partitioning method: manual. Далее настраила отдельный раздел под /boot: выбрала первый диск и создала на нем новую таблицу разделов, где Partition size: 512M; Mount point: /boot.
Повторила настройку для второго диска, но поскольку одновременно монтировать 2 раза /boot нельзя, то выбрала mount point: none. В итоге получила следующее:

![](https://github.com/Korusha/OS/blob/master/lab2/screens/1.jpg "1 screen")

#### 2. Настройка RAID.
Выбрала свободное место на первом диске и настроила в качестве типа раздела physical volume for RAID, затем выбрала "Done setting up the partition", повторила точно такую же настройку для второго диска, в результате:

![](https://github.com/Korusha/OS/blob/master/lab2/screens/2.jpg "2 screen")

Затем выбрала пункт "Configure software RAID", где 
* Create MD device 
* Software RAID device type - выбрала зеркальный массив
* Active devices for the RAID XXXX array - выбрала оба диска
* Spare devices - оставила 0 по умолчанию
* Active devices for the RAID XX array - выбрала разделы, которые созданы под raid
* Finish

![](https://github.com/Korusha/OS/blob/master/lab2/screens/3.jpg "3 screen")

#### 3. Настроила LVM в точности так, как указано в описании лабораторной работы, и получила следующую картину:

![](https://github.com/Korusha/OS/blob/master/lab2/screens/5.jpg "5 screen")

#### 4. Разметка разделов. 
По очереди выбрала каждый созданный в LVM том и разметила их.

![](https://github.com/Korusha/OS/blob/master/lab2/screens/6.jpg "6 screen")

#### 5. Закончила установку ОС, поставила grub на первое устройство (sda) и загрузила систему. 
Выполнила копирование содержимого раздела /boot с диска sda (ssd1) на диск sdb (ssd2) с помощью команды <br> `dd if=/dev/sda1 of=/dev/sdb1`

Выполнила установку grub на второе устройство, посмотрела диски в системе: <br> `fdisk -l` <br> `lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT`, где последняя показала 2 диска на 4GB и информацию о них.

Выполнила установку: `grub-install /dev/sdb`

Просмотрела информацию о текущем raid командой `cat /proc/mdstat`, где увидела активный RAID1 на sda2 и sdb2, и выводы команд: `pvs`, `vgs`, `lvs`, `mount`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/9.jpg "9 screen")
![](https://github.com/Korusha/OS/blob/master/lab2/screens/10.jpg "10 screen")

После выполнения задания 1 у меня появилась виртуальная машина с дисками ssd1, ssd2, а также настроенный RAID и LVM.

## Задание 2. Эмуляция отказа одного из дисков
#### 1. Удаление дисков на лету. Оно мне доступно, так как я поставила галочку hot swap. <br>
Я выполнила удаление диска ssd1 в свойствах машины, нашла директорию, где хранятся файлы моей виртуальной машины и удалила ssd1.vmdk. <br><br>
Убедилась, что виртуальная машина по-прежнему работает и выполнила перезагрузку виртуальной машины.<br><br>
Проверила статус RAID-массива после отказа одного из дисков: `cat /proc/mdstat`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/11.jpg "11 screen")

#### 2. Добавляю в интерфейсе VM новый диск такого же размера (4GB) и назвала его ssd3.<br>
Посмотрела, что новый диск "приехал" в систему командой `lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/13.jpg "13 screen")

#### 3. Скопировала таблицу разделов со старого диска на новый: `sfdisk -d /dev/sda | sfdisk /dev/sdb`<br>
Посмотрела результат командой `fdisk -l`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/14.jpg "14 screen")

#### 3. Добавила в рейд массив новый диск: `mdadm --manage /dev/md0 --add /dev/sdb2` <br>
Посмотрела результат `cat /proc/mdstat` и увидела, что началась синхронизация <br><br>
Вручную выполнила синхронизацию разделов, не входящих в RAID. Для этого скопировала с "живого" диска на новенький, который недавно поставила `dd if=/dev/sda1 of=/dev/sdb1`<br><br>
После завершения синхронизации установила grub на новый диск.

![](https://github.com/Korusha/OS/blob/master/lab2/screens/18.jpg "18 screen")

Выполнили перезагрузку ВМ и убедились, что все работает `cat /proc/mdstat` и `lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/19.jpg "19 screen")

После выполнения задания 2, был удален диск ssd1, сохранен диск ssd2, добавлен диск ssd3, где последний был добавлен в RAID-массив и синхронизирован.

## Задание 3. Добавление новых дисков и перенос раздела
