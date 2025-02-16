<b>Домашнее задание 03: Физический уровень PostgreSQL</b>

Цель:
создавать дополнительный диск для уже существующей виртуальной машины, размечать его и делать на нем файловую систему
переносить содержимое базы данных PostgreSQL на дополнительный диск
переносить содержимое БД PostgreSQL между виртуальными машинами

Описание/Пошаговая инструкция выполнения домашнего задания:
создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в ЯО/Virtual Box/докере
поставьте на нее PostgreSQL 15 через sudo apt
проверьте что кластер запущен через sudo -u postgres pg_lsclusters
зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
postgres=# create table test(c1 text);
postgres=# insert into test values('1');
\q
остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop
создайте новый диск к ВМ размером 10GB
добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)
сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
напишите получилось или нет и почему
задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
напишите что и почему поменяли
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
напишите получилось или нет и почему
зайдите через через psql и проверьте содержимое ранее созданной таблицы
задание со звездочкой *: не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.

# Выполнение
Создана ВМ в Яндекс облаке, сгенерирован SSH ключ.


Установка Postgresql:

sudo apt-get update

sudo apt-get -y install postgresql

sudo pg_lsclusters 

Зашли из под пользователя postgres в psql:

sudo -u postgres psql

Создал таблицу и заполнил тестовым значением:
create table test (c1 text);
insert into test values('1');



Просмотрел файл postgresql.conf:

sudo nano /etc/postgresql/16/main/postgresql.conf

Проверил куда ссылается параметр data_directory:

data_directory = '/var/lib/postgresql/16/main'



При создании ВМ сразу указал 2 диска:

Новый диск: vdb



<br><b>Далее из замечаний по ДЗ:</b>

<br>5) добавляем диск и смотрим, есть ли он
<br>sudo parted -l | grep Error - должна появиться ошибка с нераспознанной меткой диска
<br>6) проверяем, что диск виден, но не размечен
<br>lsblk
<br>7) форматируем и размечаем диск
<br>sudo parted /dev/vdb mklabel gpt
<br>sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%
<br>sudo mkfs.ext4 -L datapartition /dev/vdb1
<br>8) проверяем, что раздел создался и с диском все ок:
<br>sudo lsblk -o NAME,FSTYPE,LABEL,UUID | grep -v loop
<br>9) создаем на диске папку и монтируем этот диск
<br>sudo mkdir -p /mnt/data
<br>sudo mount -a
<br>10) перезагружаемся
<br>sync; sudo reboot
<br>11) после перезагрузки проверяем, что с диском все еще все ок:
<br>df -h /mnt/data
<br>![01_](https://github.com/user-attachments/assets/9896eea6-a158-4398-8f43-635f49e92a6b)


<br>12) выдаем права пользователю postgres
<br>sudo chown -R postgres:postgres /mnt/data/
<br>13) перемещаем все в /mnt/data
<br>sudo mv /var/lib/postgresql/16 /mnt/data
<br>14) редактируем postgresql.conf
<br>15) запускаем сервис postgresql и сам кластер:
<br>sudo systemctl start postgresql.service
<br>sudo -u postgres pg_ctlcluster 16 main start

![03_1](https://github.com/user-attachments/assets/ebc0394d-b9ce-44da-8b7d-7ab3138c9eed)


<br><b>Не получилось. вышла ошибка:</b>


otus@compute-vm-2-2-10-ssd-1739715327843:~$ sudo -u postgres pg_ctlcluster 16 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@16-main
Error: /usr/lib/postgresql/16/bin/pg_ctl /usr/lib/postgresql/16/bin/pg_ctl start -D /mnt/data -l /var/log/postgresql/postgresql-16-main.log -s -o  -c config_file="/etc/postgresql/16/main/postgresql.conf"  exited with status 1:
pg_ctl: directory "/mnt/data" is not a database cluster directory



<br><b>Параметр data_directory в postgresql.conf:</b>
<br>data_directory = '/mnt/data'

![02_](https://github.com/user-attachments/assets/eca86d07-9dc5-4e9c-aa41-51fef11b888f)


<br><b>содержимое /mnt/data:</b>

![04_](https://github.com/user-attachments/assets/2469ce42-5dd0-4fd4-af47-cd6b862e09d4)



<br><b>Стал разбираться дальше и сделал команды:</b>
<br>Создал папку: /mnt/data/postgresql/16/main
<br>sudo mkdir -p /mnt/data/postgresql/16/main

<br>Дал доступ:
<br>sudo chown -R postgres:postgres /mnt/data/postgresql


<br>Создал новую директорию данных, через команду команду initdb:
<br>sudo -u postgres /usr/lib/postgresql/16/bin/initdb -D /mnt/data/postgresql/16/main


<br>Остановил кластер:
<br>sudo service postgresql stop

<br>Изменил путь к директории данных в файле postgresql.conf
<br>data_directory='/mnt/data/postgresql/16/main'

<br>Запустил службу
<br>sudo service postgresql start

<br><b>Кластер стартанул</b>
<br>![05_](https://github.com/user-attachments/assets/f3222518-094f-4046-bafb-243d65a1e096)


<br><b>Но тестовой таблицы уже нет</b>
<br>Посмотреть доступы схемы:
<br>postgres=# \dn+
<br>
 
 public | pg_database_owner | pg_database_owner=UC/pg_database_owner+| standard public schema


<br>Попытался дать доступ на схему:
<br>GRANT ALL ON SCHEMA public TO public;

<br><b>Но никаких таблиц уже нет</b>
<br>postgres=# \d
<br>Did not find any relations.

<br>![image](https://github.com/user-attachments/assets/f961fdf6-4c98-4e4f-89a7-7050f95dae5d)

