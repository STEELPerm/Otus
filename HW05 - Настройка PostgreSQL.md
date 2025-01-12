<b>Домашнее задание 05: Настройка PostgreSQL</b>

Цель:

сделать нагрузочное тестирование PostgreSQL
настроить параметры PostgreSQL для достижения максимальной производительности

Описание/Пошаговая инструкция выполнения домашнего задания:

развернуть виртуальную машину любым удобным способом
поставить на неё PostgreSQL 15 любым способом
настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины
нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему


# Выполнение
Создана ВМ в Яндекс облаке, сгенерирован SSH ключ. Подключение к ВМ и установка на нее PostgreSQL 17:

sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql

pg_lsclusters

![01 Установка Postgresql](https://github.com/user-attachments/assets/ac7079ed-d581-476c-9ba2-5eb123545412)


Проверка параметров и наличие утилиты pgbench 

![02 Проверка параметров и наличие утилиты pgbench](https://github.com/user-attachments/assets/c695b06d-d304-4d8d-b5eb-c20c21fd8e32)

Инициализация pgbench

![03 Инициализация теста pgbench](https://github.com/user-attachments/assets/baaf97bd-e5e2-4f4a-83a3-d136a0f52308)


<br><b>Запускаем 1-й тест на 30 секунд:</b>
<br>pgbench -U postgres -h 0.0.0.0 -T 30 test
<br>Результат:
<br>latency average = 3.434 ms
<br>initial connection time = 12.509 ms
<br><b>tps = 291.180023 (without initial connection time)</b>
<br>
![04 Запуск 30 секундного теста](https://github.com/user-attachments/assets/ca7a4797-d99e-447e-bbf4-bd94281499b3)


<br><b>Запускаем 1-й тест на 30 секунд после добавления дополнительных настроек:</b>
<br>pgbench -U postgres -h 0.0.0.0 -T 30 test
<br>Результат:
<br>latency average = 3.483 ms
<br>initial connection time = 12.556 ms
<br><b>tps = 287.082169 (without initial connection time)</b>
<br><b>Настройки:</b>
<br>-- Memory Configuration
<br>ALTER SYSTEM SET shared_buffers TO '1GB';
<br>ALTER SYSTEM SET effective_cache_size TO '3GB';
<br>ALTER SYSTEM SET work_mem TO '14MB';
<br>ALTER SYSTEM SET maintenance_work_mem TO '205MB';
<br>
<br>-- Checkpoint Related Configuration
<br>ALTER SYSTEM SET min_wal_size TO '2GB';
<br>ALTER SYSTEM SET max_wal_size TO '3GB';
<br>ALTER SYSTEM SET checkpoint_completion_target TO '0.9';
<br>ALTER SYSTEM SET wal_buffers TO '-1';
<br>
<br>-- Network Related Configuration
<br>ALTER SYSTEM SET listen_addresses TO '*';
<br>ALTER SYSTEM SET max_connections TO '100';
<br>
<br>-- Storage Configuration
<br>ALTER SYSTEM SET random_page_cost TO '1.1';
<br>ALTER SYSTEM SET effective_io_concurrency TO '200';
<br>
<br>-- Worker Processes Configuration
<br>ALTER SYSTEM SET max_worker_processes TO '8';
<br>ALTER SYSTEM SET max_parallel_workers_per_gather TO '2';
<br>ALTER SYSTEM SET max_parallel_workers TO '2';
<br>
![06 Тест после добавления дополнительных настроек](https://github.com/user-attachments/assets/3d03b5d1-02ba-4023-841a-f7b7417a28d1)



<br><b>Запускаем повторно 1-й тест после восстановления настроек:</b>
<br>alter system reset all;
<br>pgbench -U postgres -h 0.0.0.0 -T 30 test
<br>Результат:
<br>latency average = 3.844 ms
<br>initial connection time = 12.436 ms
<br><b>tps = 260.158390 (without initial connection time)</b>
<br>
![07 Тест после восстановления всех настроек](https://github.com/user-attachments/assets/5266fb03-1ca8-448f-9e97-b8c3e18feeef)

<br><b>Результаты 1-го теста примерно сопоставимы.</b>




<br><b>Запускаем 2-й расширенный тест с новыми настройками:</b>
<br>pgbench -U postgres -h 0.0.0.0 -c 50 -j 2 -P 10 -T 60 test
<br>Результат:
<br>duration: 60 s
<br>number of transactions actually processed: 17509
<br>number of failed transactions: 0 (0.000%)
<br>latency average = 170.463 ms
<br>latency stddev = 233.724 ms
<br>initial connection time = 519.860 ms
<br><b>tps = 292.231109 (without initial connection time)</b>
<br><b>Настройки:</b>
<br>-- Memory Configuration
<br>ALTER SYSTEM SET shared_buffers TO '4GB';
<br>ALTER SYSTEM SET effective_cache_size TO '12GB';
<br>ALTER SYSTEM SET work_mem TO '57MB';
<br>ALTER SYSTEM SET maintenance_work_mem TO '819MB';
<br>
<br>-- Checkpoint Related Configuration
<br>ALTER SYSTEM SET min_wal_size TO '2GB';
<br>ALTER SYSTEM SET max_wal_size TO '3GB';
<br>ALTER SYSTEM SET checkpoint_completion_target TO '0.9';
<br>ALTER SYSTEM SET wal_buffers TO '-1';
<br>
<br>-- Network Related Configuration
<br>ALTER SYSTEM SET listen_addresses TO '*';
<br>ALTER SYSTEM SET max_connections TO '100';
<br>
<br>-- Storage Configuration
<br>ALTER SYSTEM SET random_page_cost TO '1.1';
<br>ALTER SYSTEM SET effective_io_concurrency TO '200';
<br>
<br>-- Worker Processes Configuration
<br>ALTER SYSTEM SET max_worker_processes TO '8';
<br>ALTER SYSTEM SET max_parallel_workers_per_gather TO '2';
<br>ALTER SYSTEM SET max_parallel_workers TO '2';
<br>
![08_1 Проведение 3го теста с новыми настройками](https://github.com/user-attachments/assets/03cc82c0-6e97-42a6-89fa-fe49dd9c0786)


<br><b>Запускаем 2-й расширенный тест со стандартными настройками:</b>
<br>pgbench -U postgres -h 0.0.0.0 -c 50 -j 2 -P 10 -T 60 test
<br>Результат:
<br>duration: 60 s
<br>number of transactions actually processed: 14900
<br>number of failed transactions: 0 (0.000%)
<br>latency average = 200.027 ms
<br>latency stddev = 273.561 ms
<br>initial connection time = 515.139 ms
<br><b>tps = 249.443649 (without initial connection time)</b>
<br>
![08_2 Проведение 3го теста со стандартными настройками](https://github.com/user-attachments/assets/4029d193-ffd0-4001-8342-228ea5b21566)


<br><b>Результаты 2-го теста лучше с дополнительными настройкаим.</b>
<br>
<br>
<br>Настройки были сгенерированны на сайте https://www.pgconfig.org
<br>
![10 настройки с сайта pgconfig org](https://github.com/user-attachments/assets/ab8ebba5-557d-4f2a-95ab-3013e10a6020)

<br><br>Данные ВМ:
<br>
![11 настройки виртуальной машины](https://github.com/user-attachments/assets/b60d65ac-c87f-49bd-899c-60da66ea8fe3)

<br><br>Дашборд с нагрузками к базе:
<br>
![09 Дашборд с нагрузками](https://github.com/user-attachments/assets/f51cc3a7-2390-4b53-8ec4-69905c551a16)

