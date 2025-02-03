# Название выполняемого задания;
Практические навыки работы с ZFS

# Текст задания
Что нужно сделать?

Определить алгоритм с наилучшим сжатием:
Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
создать 4 файловых системы на каждой применить свой алгоритм сжатия;
для сжатия использовать либо текстовый файл, либо группу файлов.
Определить настройки пула.
С помощью команды zfs import собрать pool ZFS.
Командами zfs определить настройки:
   
- размер хранилища;
    
- тип pool;
    
- значение recordsize;
   
- какое сжатие используется;
   
- какая контрольная сумма используется.
Работа со снапшотами:
скопировать файл из удаленной директории;
восстановить файл локально. zfs receive;
найти зашифрованное сообщение в файле secret_message.

## Часть 1

То что имеем на старте

```bash
vagrant@belousovZfs:~$ lsblk
NAME                      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0  64G  0 disk 
├─sda1                      8:1    0   1M  0 part 
├─sda2                      8:2    0   2G  0 part /boot
└─sda3                      8:3    0  62G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0  31G  0 lvm  /
sdb                         8:16   0   1G  0 disk 
sdc                         8:32   0   1G  0 disk 
sdd                         8:48   0   1G  0 disk 
sde                         8:64   0   1G  0 disk 
sdf                         8:80   0   1G  0 disk 
sdg                         8:96   0   1G  0 disk 
sdh                         8:112  0   1G  0 disk 
sdi                         8:128  0   1G  0 disk 
```
Создаем сразу 4 пула

```bash
vagrant@belousovZfs:~$ sudo zpool create otus1 mirror /dev/sdb /dev/sdc && sudo zpool create otus2 mirror /dev/sdd /dev/sde && sudo zpool create otus3 mirror /dev/sdf /dev/sdg && sudo zpool create otus4 mirror /dev/sdh /dev/sdi
```

Просматриваем что все получилось
```bash
vagrant@belousovZfs:~$ sudo zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   960M   108K   960M        -         -     0%     0%  1.00x    ONLINE  -
otus2   960M   116K   960M        -         -     0%     0%  1.00x    ONLINE  -
otus3   960M   116K   960M        -         -     0%     0%  1.00x    ONLINE  -
otus4   960M   122K   960M        -         -     0%     0%  1.00x    ONLINE  -
```

Задаем разные алгоритмы сжатия
```bash
vagrant@belousovZfs:~$ sudo zfs set compression=lzjb otus1 && sudo zfs set compression=lz4 otus2 && sudo zfs set compression=gzip-9 otus3 && sudo zfs set compression=zle otus4
```

Убеждаемся что все норм
```bash
vagrant@belousovZfs:~$ sudo zfs get all | grep compression
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local
```

Заливаем в каждый файл
```bash
vagrant@belousovZfs:~$ for i in {1..4}; do sudo wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
--2025-02-03 16:32:10--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41123477 (39M) [text/plain]
Saving to: ‘/otus1/pg2600.converter.log’

pg2600.converter.log                        100%[========================================================================================>]  39.22M  6.34MB/s    in 6.9s    

2025-02-03 16:32:17 (5.69 MB/s) - ‘/otus1/pg2600.converter.log’ saved [41123477/41123477]

--2025-02-03 16:32:17--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41123477 (39M) [text/plain]
Saving to: ‘/otus2/pg2600.converter.log’

pg2600.converter.log                        100%[========================================================================================>]  39.22M  8.39MB/s    in 5.8s    

2025-02-03 16:32:24 (6.73 MB/s) - ‘/otus2/pg2600.converter.log’ saved [41123477/41123477]

--2025-02-03 16:32:24--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41123477 (39M) [text/plain]
Saving to: ‘/otus3/pg2600.converter.log’

pg2600.converter.log                        100%[========================================================================================>]  39.22M  8.37MB/s    in 5.6s    

2025-02-03 16:32:30 (7.05 MB/s) - ‘/otus3/pg2600.converter.log’ saved [41123477/41123477]

--2025-02-03 16:32:30--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41123477 (39M) [text/plain]
Saving to: ‘/otus4/pg2600.converter.log’

pg2600.converter.log                        100%[========================================================================================>]  39.22M  7.72MB/s    in 6.1s    

2025-02-03 16:32:37 (6.44 MB/s) - ‘/otus4/pg2600.converter.log’ saved [41123477/41123477]
```

Проверяем что все залилось

```bash
vagrant@belousovZfs:~$ ls -l /otus*
/otus1:
total 22096
-rw-r--r-- 1 root root 41123477 Feb  2 08:31 pg2600.converter.log

/otus2:
total 18006
-rw-r--r-- 1 root root 41123477 Feb  2 08:31 pg2600.converter.log

/otus3:
total 10966
-rw-r--r-- 1 root root 41123477 Feb  2 08:31 pg2600.converter.log

/otus4:
total 40188
-rw-r--r-- 1 root root 41123477 Feb  2 08:31 pg2600.converter.log
```

Анализируем занятое место
```bash
vagrant@belousovZfs:~$ sudo zfs list
NAME    USED  AVAIL  REFER  MOUNTPOINT
otus1  21.7M   810M  21.6M  /otus1
otus2  17.7M   814M  17.6M  /otus2
otus3  10.9M   821M  10.7M  /otus3
otus4  39.4M   793M  39.3M  /otus4
```

```bash
vagrant@belousovZfs:~$ sudo zfs get all | grep compressratio | grep -v ref
otus1  compressratio         1.81x                  -
otus2  compressratio         2.23x                  -
otus3  compressratio         3.65x                  -
otus4  compressratio         1.00x                  -
```

## Часть 2

Затягиваем архив

```bash
vagrant@belousovZfs:~$ wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
--2025-02-03 16:40:52--  https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download
Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 209.85.233.132, 2a00:1450:4010:c03::84
Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|209.85.233.132|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7275140 (6.9M) [application/octet-stream]
Saving to: ‘archive.tar.gz’

archive.tar.gz                             100%[========================================================================================>]   6.94M  11.2MB/s    in 0.6s    

2025-02-03 16:41:02 (11.2 MB/s) - ‘archive.tar.gz’ saved [7275140/7275140]
```

Распоковываем
```bash
vagrant@belousovZfs:~$ tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
```

Проверяем
```bash
vagrant@belousovZfs:~$ sudo zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
status: Some supported features are not enabled on the pool.
	(Note that they may be intentionally disabled if the
	'compatibility' property is set.)
 action: The pool can be imported using its name or numeric identifier, though
	some features will not be available without an explicit 'zpool upgrade'.
 config:

	otus                                 ONLINE
	  mirror-0                           ONLINE
	    /home/vagrant/zpoolexport/filea  ONLINE
	    /home/vagrant/zpoolexport/fileb  ONLINE
```

Импортируем
```bash
vagrant@belousovZfs:~$ sudo zpool import -d zpoolexport/ otus
```

Проверяем статус
```bash
vagrant@belousovZfs:~$ sudo zpool status otus
  pool: otus
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
	The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
	the pool may no longer be accessible by software that does not support
	the features. See zpool-features(7) for details.
config:

	NAME                                 STATE     READ WRITE CKSUM
	otus                                 ONLINE       0     0     0
	  mirror-0                           ONLINE       0     0     0
	    /home/vagrant/zpoolexport/filea  ONLINE       0     0     0
	    /home/vagrant/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
```

Проверяем значение

```bash
vagrant@belousovZfs:~$ sudo zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
```

```bash
vagrant@belousovZfs:~$ sudo  zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     defaul
```

```bash
vagrant@belousovZfs:~$ sudo  zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
```

```bash
vagrant@belousovZfs:~$ sudo zfs get compression otus
NAME  PROPERTY     VALUE           SOURCE
otus  compression  zle             local
```

```bash
vagrant@belousovZfs:~$ sudo zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
```

## Часть 3

Скачиваем снимок
```bash 
vagrant@belousovZfs:~$ wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
[1] 5785
vagrant@belousovZfs:~$ 
Redirecting output to ‘wget-log’.
```

Накатываем его
```bash
vagrant@belousovZfs:~$ sudo zfs receive otus/test@today < otus_task2.file
```

Проверяем результат
```bash
vagrant@belousovZfs:~$ cat /otus/test/task1/file_mess/secret_message
https://otus.ru/lessons/linux-hl/
```

И вот оно секретное значение ) https://otus.ru/lessons/linux-hl/