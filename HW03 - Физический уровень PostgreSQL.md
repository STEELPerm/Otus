<b>Домашнее задание 03: Установка и настройка PostgreSQL</b>

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


Далее остановил postgres через:

sudo -u postgres pg_ctlcluster 17 main stop:
![3 Остановили postgres через sudo -u postgres pg_ctlcluster 17 main stop](https://github.com/user-attachments/assets/1e981566-d2d6-4f9f-9dff-c0e0c7e4dc8a)

Далее на YC создал и подключил диск:
![4 Создали диск](https://github.com/user-attachments/assets/ec71a7ac-bf98-4d58-949d-f8bdd1665758)
![4_1 Подключили диск](https://github.com/user-attachments/assets/a108e88b-d985-4622-ab88-96fc728f282f)

Далее перезапустил кластер:
sudo -u postgres pg_ctlcluster 17 main start
![5  Перезапустил кластер - sudo -u postgres pg_ctlcluster 17 main start](https://github.com/user-attachments/assets/4ed2637c-f50b-475d-81c9-03600fb0a7e0)


Далее зашёл через psql и проверил содержимое ранее созданной таблицы:
![6 Зашёл через psql и проверил содержимое ранее созданной таблицы](https://github.com/user-attachments/assets/71afd37b-a79c-45a0-ae53-f5b08cfe79bf)

