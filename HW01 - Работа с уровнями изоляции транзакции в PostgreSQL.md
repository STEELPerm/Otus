<b>Домашнее задание 01: Работа с уровнями изоляции транзакции в PostgreSQL</b>

Цель:
научиться работать с Google Cloud Platform на уровне Google Compute Engine (IaaS)
научиться управлять уровнем изоляции транзации в PostgreSQL и понимать особенность работы уровней read commited и repeatable read

Описание/Пошаговая инструкция выполнения домашнего задания:
создать новый проект в Яндекс облако или на любых ВМ, докере
далее создать инстанс виртуальной машины с дефолтными параметрами
добавить свой ssh ключ в metadata ВМ
зайти удаленным ssh (первая сессия), не забывайте про ssh-add
поставить PostgreSQL
зайти вторым ssh (вторая сессия)
запустить везде psql из под пользователя postgres
выключить auto commit
сделать

в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

посмотреть текущий уровень изоляции: show transaction isolation level
начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
сделать select from persons во второй сессии
видите ли вы новую запись и если да то почему?
завершить первую транзакцию - commit;
сделать select from persons во второй сессии
видите ли вы новую запись и если да то почему?
завершите транзакцию во второй сессии
начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
сделать select* from persons во второй сессии*
видите ли вы новую запись и если да то почему?
завершить первую транзакцию - commit;
сделать select from persons во второй сессии
видите ли вы новую запись и если да то почему?
завершить вторую транзакцию
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?

# Выполнение

Создана ВМ в Яндекс облаке, сгенерирован SSH ключ. Подключение к ВМ через Putty и установка PostgreSQL.
![HM01_1](https://github.com/user-attachments/assets/40189f35-5d9e-43df-abe8-ddfe2390765a)

Запущено 2 сессии, создана база my_db и произведено подключение к ней.
<br><br><b>Далее по заданию:</b>
<br>•	выключить auto commit
<br>
![HM01_2](https://github.com/user-attachments/assets/08b0085a-82e1-4cb9-a4b2-4e543f77b2ac)

<br><b>Далее по заданию:</b>
<br>в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
<br>•	посмотреть текущий уровень изоляции: show transaction isolation level
<br>•	начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
<br>•	в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
<br>•	сделать select from persons во второй сессии
<br>•	видите ли вы новую запись и если да то почему?
<br><b>ОТВЕТ: Во второй сессии новая запись не видна, т.к. транзакция в первой сессии ещё не завершилась.</b> 
<br>
![HM01_3](https://github.com/user-attachments/assets/16d66ebf-5312-4821-938a-3cd86e514ae1)

<br><b>Далее по заданию:</b>
<br>•	завершить первую транзакцию - commit;
<br>•	сделать select from persons во второй сессии
<br>•	видите ли вы новую запись и если да то почему?
<br><b>ОТВЕТ: Во второй сесиии новая запись видна, т.к. в первой сесии завершили транзакцию.</b>
![HM01_4](https://github.com/user-attachments/assets/d9a0068c-03ea-4916-bd72-8bc62d83bc97)

<br><b>Далее по заданию:</b>
<br>•	завершите транзакцию во второй сессии
<br>•	начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
<br>•	в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
<br>•	сделать select* from persons во второй сессии*
<br>•	видите ли вы новую запись и если да то почему?
<br><b>ОТВЕТ: Новая запись не видна, т.к. транзакция в первой сессии ещё не завершилась.</b>
![HM01_5](https://github.com/user-attachments/assets/ba6329f5-c7c7-4b1c-ace1-079c4e7e6ec3)

<br><b>Далее по заданию:</b>
<br>•	завершить первую транзакцию - commit;
<br>•	сделать select from persons во второй сессии
<br>•	видите ли вы новую запись и если да то почему?
<br><b>ОТВЕТ: Во второй сессии новая запись не видна, т.к. в ней выставлен уровень изоляции «repeatable read» - повторное чтение.</b>
![HM01_6](https://github.com/user-attachments/assets/9ae3f91f-d9c9-497b-a8e7-5a2e91050fae)

<br><b>Далее по заданию:</b>
<br>•	завершить вторую транзакцию
<br>•	сделать select * from persons во второй сессии
<br>•	видите ли вы новую запись и если да то почему?
<br><b>ОТВЕТ: Во втором сеансе новая запись видна, т.к. мы завершили вторую транзакцию.</b>
![HM01_7](https://github.com/user-attachments/assets/1652d27f-b72c-4e30-8aa1-df35b110cf29)



