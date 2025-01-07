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
Создана ВМ в Яндекс облаке, сгенерирован SSH ключ. Подключение к ВМ через Putty и установка на нее PostgreSQL 17 через sudo apt:

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

sudo apt-get update
sudo apt-get -y install postgresql
sudo pg_lsclusters 

![1 Установка PostgreSQL через sudo apt, проверка, что кластер запущен через sudo -u postgres pg_lsclusters](https://github.com/user-attachments/assets/07af4baf-fc6e-4806-803b-19c913fc614a)

Далее зашёл из под пользователя postgres в psql, создал таблицу test, заполнил таблицу 1 значением:

![2 Зашли из под пользователя postgres в psql, создали таблицу test, заполнили таблицу 1 значением](https://github.com/user-attachments/assets/58103714-2e55-4247-a4a6-e5cb1ee944f6)


Просмотрел файл postgresql.conf:

sudo nano /etc/postgresql/17/main/postgresql.conf

Проверил куда ссылается параметр data_directory:

data_directory = '/var/lib/postgresql/17/main'

![555_Куда ссылается параметр data_directory](https://github.com/user-attachments/assets/38ad91a4-6ad0-4ec2-9d1a-1bbd6ea59901)



При создании ВМ сразу указал 2 диска:

Новый диск: vdb

![555_Подключено 2 диска](https://github.com/user-attachments/assets/7c7cd597-7bec-478b-b5cc-e454a2a7fd73)


Смонтировал новый диск, проверил его состояние, UID:

/dev/vdb1: UUID="eb3a25eb-d433-4a94-8444-75cfe4474153" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="primary" PARTUUID="853e152b-c497-49db-a0ae-4528a78c8165"


![555_Смонтировал новый диск, просмотрел состояние диска](https://github.com/user-attachments/assets/f2e9a480-23e8-4e3b-a025-17a818eb7f44)



Далее создал каталог mnt22 и подключил раздел vdb1 к новому каталогу mnt22:

sudo mkdir /mnt22
sudo mount /dev/vdb1 /mnt22

![555_Создал каталог mnt22 и подключил раздел vdb1 к новому каталогу mnt22](https://github.com/user-attachments/assets/d7ed6142-8d84-4da6-aa4a-43ac677e8f1d)


Далее сделал пользователя postgres владельцем mnt22:

![555_Сделал пользователя postgres владельцем mnt22](https://github.com/user-attachments/assets/19ce4ad1-2ab0-41d2-8d88-0afa28e82bfb)


Далее скопировал данные из /var/lib/postgresql/17/main в mnt22 и перезапустил кластер:

sudo mv /var/lib/postgresql/17/main /mnt22

sudo pg_ctlcluster 17 main restart

![555_Скопировал данные в mnt22 и перезапустил кластер](https://github.com/user-attachments/assets/30ead841-8f59-4a6f-bf8e-4dc9cfc7a7c3)



Далее поменял в файле postgresql.conf путь у data_directory на /mnt22

![555_Содержимое файла postgresql conf после изменения пути data_directory](https://github.com/user-attachments/assets/0a16f308-6b6f-4baf-a788-9dcedca52859)



Содержание папки mnt22 (похоже копирование не произошло): 
![555_Содержимое папки mnt22](https://github.com/user-attachments/assets/f2edeeea-1123-4da4-a0f2-4bd427911117)


Похоже, что копирование не прошло(( Хотя видно, что на новом диске стало занято больше места, и в мониторинге яндекса тоже видно, что было копирование.

![555_Мониторинг нового диска после копирования](https://github.com/user-attachments/assets/92e3dcfd-0598-4c14-9345-73520ac802de)

![555_Занято места на новом диске](https://github.com/user-attachments/assets/1007de90-0890-4803-bff3-51fdd2aeb51c)

