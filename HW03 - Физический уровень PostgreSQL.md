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
```
sudo apt-get -y install postgresql
```
Зашли из под пользователя postgres в psql:
```
sudo -u postgres psql
```
Создал таблицу и заполнил тестовым значением:
```
create table test (c1 text);
insert into test values('1');
```


Просмотрел файл postgresql.conf:
```
sudo nano /etc/postgresql/16/main/postgresql.conf
```
Проверил куда ссылается параметр data_directory:
```
data_directory = '/var/lib/postgresql/16/main'
```


При создании ВМ сразу указал 2 диска:

Новый диск: vdb


<br><b>Далее из замечаний по ДЗ:</b>

<br>5) добавляем диск и смотрим, есть ли он
```
sudo parted -l | grep Error - должна появиться ошибка с нераспознанной меткой диска
```
<br>6) проверяем, что диск виден, но не размечен
```
lsblk
```
<br>7) форматируем и размечаем диск
```
sudo parted /dev/vdb mklabel gpt
sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%
sudo mkfs.ext4 -L datapartition /dev/vdb1
```
<br>8) проверяем, что раздел создался и с диском все ок:
```
sudo lsblk -o NAME,FSTYPE,LABEL,UUID | grep -v loop
```
<br>9) создаем на диске папку и монтируем этот диск
```
sudo mkdir -p /mnt/data
sudo mount -a
```
<br>10) перезагружаемся
```
sync; sudo reboot
```
<br>11) после перезагрузки проверяем, что с диском все еще все ок:
```
df -h /mnt/data
```
<br>![01_1](https://github.com/user-attachments/assets/5edf8875-52d6-4cb1-ae22-bde551893ccd)

<br>12) выдаем права пользователю postgres
```
sudo chown -R postgres:postgres /mnt/data/
```
<br>13) перемещаем все в /mnt/data
```
sudo mv /var/lib/postgresql/16 /mnt/data
```
<br>14) редактируем postgresql.conf
```
sudo nano /etc/postgresql/16/main/postgresql.conf
```
<br>![01_2](https://github.com/user-attachments/assets/74d2e0a3-0751-4789-b3f0-5b715a9cf738)


<br>15) запускаем сервис postgresql и сам кластер:
```
sudo systemctl start postgresql.service
sudo -u postgres pg_ctlcluster 16 main start
```
<br>16) Подключаемся к БД и проверяем данные.
<br><b>Всё хорошо. Таблица и данные присутствуют:</b>
<br>![01_3](https://github.com/user-attachments/assets/5d8f0987-d89e-43dd-a10f-caa94630e38a)




