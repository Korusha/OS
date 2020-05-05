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

#### 4. Добавила в рейд массив новый диск: `mdadm --manage /dev/md0 --add /dev/sdb2` <br>
Посмотрела результат `cat /proc/mdstat` и увидела, что началась синхронизация <br><br>
Вручную выполнила синхронизацию разделов, не входящих в RAID. Для этого скопировала с "живого" диска на новенький, который недавно поставила `dd if=/dev/sda1 of=/dev/sdb1`<br><br>
После завершения синхронизации установила grub на новый диск.

![](https://github.com/Korusha/OS/blob/master/lab2/screens/18.jpg "18 screen")

Выполнили перезагрузку ВМ и убедились, что все работает `cat /proc/mdstat` и `lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/19.jpg "19 screen")

После выполнения задания 2, был удален диск ssd1, сохранен диск ssd2, добавлен диск ssd3, где последний был добавлен в RAID-массив и синхронизирован.

## Задание 3. Добавление новых дисков и перенос раздела
#### 1. Проэмулировала отказ диска ssd2, удалив из свойств ВМ диск и перезагрузившись
Посмотрела текущее состояние дисков и RAID:<br>
`cat /proc/mdstat`<br>
`fdisk -l`<br>
`lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT`<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/20.jpg "20 screen")

![](https://github.com/Korusha/OS/blob/master/lab2/screens/21.jpg "21 screen")

#### 2. Добавила один новый SSD диск, назвав его ssd4, объем которого 5 GB.
Проверила, что произошло `lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT`: 

![](https://github.com/Korusha/OS/blob/master/lab2/screens/22.jpg "22 screen")

Cкопировала файловую таблицу со старого диска на новый:<br>
`sfdisk -d /dev/sda | sfdisk /dev/sdb`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/23.jpg "23 screen")

С помощью команды dd скопировала данные /boot на новый диск:
`dd if=/dev/sda1 of=/dev/sdb1`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/25.jpg "25 screen")

Так как /boot остался смонтирован на старом диске, перемонтировала его на живой диск:<br>
* `mount | grep boot` # посмотрела, куда смонтирован диск<br>
* `lsblk` # посмотрела, какие диски есть в системе, и есть ли диск, полученный из предыдущего пункта<br>
* `umount /boot` # отмонтировала /boot<br>
* `mount -a` # выполнила монтирование всех точек согласно /etc/fstab. <br>

Также установила загрузчик на новый ssd диск:<br>
`grub-install /dev/sdb`

Затем создала новый рейд-массив с включением только одного нового ssd диска, использовав специальный ключ --force:<br>
`mdadm --create --verbose /dev/md63 --level=1 --force --raid-devices=1 /dev/sdb2`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/26.jpg "26 screen")

С помощью команды `cat /proc/mdstat` проверила результат операции, где мы увидели новый RAID массив на sdb. И посмотрела информацию о дисках и RAID с помощью `lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/27.jpg "27 screen")

#### 3. Настраиваю LVM
* Выполнила команду `pvs` для просмотра информации о текущих физических томах
* Создала новый физический том, включив в него ранее созданный RAID массив: `pvcreate /dev/md63`
* Выполнила команду `lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT`
* Снова выполнила команду `pvs`, чтобы увидеть новую информацию

Увеличила размер Volume Group System с помощью команды:<br>
`vgextend system /dev/md63`<br>
Выполнила команды:
* `vgdisplay system -v`
* `pvs`
* `vgs`
* `lvs -a -o+devices`<br>
Информация о физических томах после увеличения размера Volume Group system.

![](https://github.com/Korusha/OS/blob/master/lab2/screens/28.jpg "28 screen")<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/29.jpg "29 screen")

LV var,log,root все еще находятся на md0.<br>
Выполнила перемещение данных со старого диска на новый, повторив операцию для всех logical volume: <br>
* `pvmove -i 10 -n /dev/system/root /dev/md0 /dev/md63`<br>
* `pvmove -i 10 -n /dev/system/var /dev/md0 /dev/md63`<br>
* `pvmove -i 10 -n /dev/system/log /dev/md0 /dev/md63`<br>

![](https://github.com/Korusha/OS/blob/master/lab2/screens/30.jpg "30 screen")<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/31.jpg "31 screen")<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/32.jpg "32 screen")<br>
Теперь LV var,log,root находятся на md63.

Изменяю VG, удалив из него диск старого RAID<br>
`vgreduce system /dev/md0`<br>
И смотрю информацию после удаления:<br>
* `lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT`
* `pvs`
* `vgs`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/33.jpg "33 screen")

#### 4. Удаляю ssd3 и добавляю ssd5 (5 GB), hdd1 (8 GB), hdd2 (8 GB)
Информация после добавления дисков<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/35.jpg "35 screen")

Копирую таблицу разделов на новый диск. Затем копируйте загрузочный раздел /boot с диска ssd4 на ssd5<br>
`dd if=/dev/sdb1 of=/dev/sda1`<br>
И установливаю grub на новый диск (ssd5)

![](https://github.com/Korusha/OS/blob/master/lab2/screens/37.jpg "37 screen")

Изменила размер второго раздела диска ssd5, перечитала таблицу разделов, результат: <br>

![](https://github.com/Korusha/OS/blob/master/lab2/screens/38.jpg "38 screen")

Информация о дисках после добавления нового диска к RAID массиву и расширения массива до 2-х дисков

![](https://github.com/Korusha/OS/blob/master/lab2/screens/39.jpg "39 screen")

#### 5. Увеличиваю размер раздела на диске ssd4
Затем перечитываю таблицу разделов и проверяю результат<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/40.jpg "40 screen")<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/41.jpg "41 screen")<br>
Теперь sda2, sdb2 разделы имеют размер > чем размер raid-устройства

#### 6. Расширяю RAID-массив
Для этого использую команду и затем проверяю результат:
* `mdadm --grow /dev/md127 --size=max`
* `lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT`<br>

![](https://github.com/Korusha/OS/blob/master/lab2/screens/42.jpg "42 screen")<br>

#### 7. Расширяю VG root,var,log
Смотрю, чему равен размер PV, и затем расширяю его размер. Он увеличился на 1 GB<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/43.jpg "43 screen")<br>
Добавляю вновь появившееся место VG var,root<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/44.jpg "44 screen")<br>
На этом этапе я завершила миграцию основного массива на новые диски. Работа с ssd1,ssd2 закончена.

#### 8. Перемещаю /var/log на новые диски
Для этого создаю новый массив `md125` и lvm на hdd дисках, также создаю новый PV на рейде из больших дисков и в нем группу с названием `data`, далее создаю логический том размером всего свободного пространства с названием `val_log` и форматирую созданный раздел в ext4. Результат:

![](https://github.com/Korusha/OS/blob/master/lab2/screens/46.jpg "46 screen")<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/47.jpg "47 screen")<br>

#### 9. Переношу данные логов со старого раздела на новый
![](https://github.com/Korusha/OS/blob/master/lab2/screens/48.jpg "48 screen")<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/49.jpg "49 screen")<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/50.jpg "50 screen")<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/51.jpg "51 screen")<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/52.jpg "52 screen")<br>

#### 10. Правлю /etc/fstab fstab 
` /etc/fstab fstab` - файл, в котором записываются правила, по которым при загрузке будут смонтированы разделы. Поправляю устройство `system-log` на `data-var_log`

![](https://github.com/Korusha/OS/blob/master/lab2/screens/53.jpg "53 screen")

#### 11. Финальный аккорд
Выполнила перезагрузку, так как я все сделала правильно, без проблем попала в свою ОС. Выполняю проверки, чтобы убедиться, что все, что я хотела сделать, я сделала, а именно:
* `pvs`
* `lvs`
* `vgs`
* `lsblk`
* `cat /proc/mdstat`<br>

До перезагрузки:<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/54.jpg "54 screen")<br> 
После перезагрузки:<br>
![](https://github.com/Korusha/OS/blob/master/lab2/screens/55.jpg "55 screen")<br> 
![](https://github.com/Korusha/OS/blob/master/lab2/screens/56.jpg "56 screen")
