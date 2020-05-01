# Описание лабораторной работы №2
## Задание 1.
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
