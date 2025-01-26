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

Подключились к ВМ:
ssh -i ~/yc_key otus@51.250.42.122


Установка Postgresql:

sudo apt-get update

sudo apt-get -y install postgresql

sudo pg_lsclusters 

Зашли из под пользователя postgres в psql:

sudo -u postgres psql


![01 Установка postgresql на ВМ в YC](https://github.com/user-attachments/assets/560b6118-4f44-4065-8195-81dd6adfee33)




Далее зашёл из под пользователя postgres в psql, создал таблицу test, заполнил таблицу 1 значением:

![01_1 Создали таблицу и поменяли пароль](https://github.com/user-attachments/assets/2abea28c-32f4-4fc5-b0df-2279ea555b6c)




Просмотрел файл postgresql.conf:

sudo nano /etc/postgresql/16/main/postgresql.conf

Проверил куда ссылается параметр data_directory:

data_directory = '/var/lib/postgresql/16/main'

![02 Куда ссылается параметр data_directory](https://github.com/user-attachments/assets/f52f3223-995e-47c6-ada5-6c777dd81ee0)




При создании ВМ сразу указал 2 диска:

Новый диск: vdb

![03 просмотр дисков](https://github.com/user-attachments/assets/3396d442-aba7-41b5-b22d-d6bf2330cf37)



<br><b>Из замечания по ДЗ</b>

Выполнена разметка нового раздела в файловую систему:

parted /dev/vdb mklabel gpt

parted -a optimal -s /dev/vdb mkpart primary ext4 0% 100%

mkfs.ext4 /dev/vdb1

Смонтировал новый диск, проверил его состояние, UID:

/dev/vdb1: UUID="4e711576-7dd0-4cc0-881d-ff2c32b1d909" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="primary" PARTUUID="4d3a4dee-df0b-4bf2-a44f-11b89ffd00f9"


![04 разметка нового раздела в файловую систему](https://github.com/user-attachments/assets/1e4752c4-d5bf-4858-a329-015d1264fcb8)



Далее создал каталог mnt22 и подключил раздел vdb1 к новому каталогу mnt22:

sudo mkdir /mnt22

sudo mount /dev/vdb1 /mnt22

Сделал пользователя postgres владельцем mnt22:
sudo chown -R postgres:postgres /mnt22/

![05 Создание каталога mnt22 и подключил раздел vdb1 к этому новому каталогу](https://github.com/user-attachments/assets/2a823a9b-2445-4516-bc7f-274fa5d3a89b)



Далее скопировал данные из /var/lib/postgresql/16/main в mnt22 и перезапустил кластер:

sudo mv /var/lib/postgresql/16/main /mnt22


<b>Перезапуск кластера не сработал:</b>

sudo pg_ctlcluster 16 main restart 

Error: /var/lib/postgresql/16/main is not accessible or does not exist

![06_Скопировал данные в mnt22](https://github.com/user-attachments/assets/7edd27c2-9569-4061-8b34-810a7165ac61)



Далее поменял в файле postgresql.conf путь у data_directory на /mnt22
![07 Содержимое файла postgresql conf после изменения пути data_directory](https://github.com/user-attachments/assets/34c1a6c3-df53-4c5f-ab3f-f85f55ab595d)


В итоге не заходит в postgresql:
![08 не заходит в postgresql](https://github.com/user-attachments/assets/9ec7ec0e-298c-445c-9cee-10d5c5edc74a)


Мониторинг ВМ:

![Мониторинг ВМ](https://github.com/user-attachments/assets/c3298cac-c539-43ab-af7e-cd976e071fd9)


Занято места на новом диске:
![место на диске](https://github.com/user-attachments/assets/98f6fc97-4d50-475b-9a04-702e16664c80)



