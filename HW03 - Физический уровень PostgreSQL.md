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

![01 Установка postgresql и создание таблицы](https://github.com/user-attachments/assets/23b8a02b-083f-4da7-8c48-8562587e173a)



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
![02 Проверяем что диск виден](https://github.com/user-attachments/assets/95814278-ab30-4632-b515-e9605331bf86)


<br>7) форматируем и размечаем диск
<br>sudo parted /dev/vdb mklabel gpt
<br>sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%
<br>sudo mkfs.ext4 -L datapartition /dev/vdb1
<br>8) проверяем, что раздел создался и с диском все ок:
<br>sudo lsblk -o NAME,FSTYPE,LABEL,UUID | grep -v loop

![03 форматируем и размечаем диск](https://github.com/user-attachments/assets/572b909c-f048-4ef3-89ea-7710e250b734)


<br>9) создаем на диске папку и монтируем этот диск
<br>sudo mkdir -p /mnt/data
<br>sudo mount -a
<br>10) перезагружаемся
<br>sync; sudo reboot
<br>11) после перезагрузки проверяем, что с диском все еще все ок:
<br>df -h /mnt/data

![04_1 создаем на диске папку и монтируем этот диск](https://github.com/user-attachments/assets/9e4f37c6-544a-4f8f-9bc3-5e33cc9300a4)



<br>12) выдаем права пользователю postgres
<br>sudo chown -R postgres:postgres /mnt/data/
<br>13) перемещаем все в /mnt/data
<br>sudo mv /var/lib/postgresql/16 /mnt/data
<br>14) редактируем postgresql.conf
<br>15) запускаем сервис postgresql и сам кластер:
<br>sudo systemctl start postgresql.service
<br>sudo -u postgres pg_ctlcluster 16 main start


![05_1 Переместили в Mnt и отредактировали postgresql conf](https://github.com/user-attachments/assets/34d4059d-ac2b-4e27-a60a-31bae38dd0c4)

<br><b>Не получилось. вышла ошибка:<b>
<br>directory "/mnt/data" is not a database cluster directory

<br>Пробовал перезагружать ВМ, сервис, кластер, менял data_directory на "/mnt" - всё-равно ошибка...
<br>Параметр data_directory в postgresql.conf:
<br>data_directory = '/mnt/data'

![06 не стартует кластер](https://github.com/user-attachments/assets/58d0791b-643a-4352-9b23-83b384ad6c42)

