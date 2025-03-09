<b>Домашнее задание 14: Работа с join'ами, статистикой</b>

<br>Цель:
<br>знать и уметь применять различные виды join'ов
<br>строить и анализировать план выполенения запроса
<br>оптимизировать запрос
<br>уметь собирать и анализировать статистику для таблицы

<br>Описание/Пошаговая инструкция выполнения домашнего задания:
<br>В результате выполнения ДЗ вы научитесь пользоваться различными вариантами соединения таблиц.
<br>В данном задании тренируются навыки:
<br>написания запросов с различными типами соединений
<br>Необходимо:
<br>Реализовать прямое соединение двух или более таблиц
<br>Реализовать левостороннее (или правостороннее) соединение двух или более таблиц
<br>Реализовать кросс соединение двух или более таблиц
<br>Реализовать полное соединение двух или более таблиц
<br>Реализовать запрос, в котором будут использованы разные типы соединений
<br>Сделать комментарии на каждый запрос
<br>К работе приложить структуру таблиц, для которых выполнялись соединения


# Выполнение
<br>Подключились к ВМ с Postgresql, созданной в VirtualBox, к созданной ранее базе demo. База была скачена в рамках домашнего задания Секционирование (HW10 - Секционирование.md).

<br>Для создания различных соединений использовались таблицы из базы demo. Диаграмма таблиц:
<br>![01_1](https://github.com/user-attachments/assets/fc9a0693-9800-41a5-8aa7-0ef1db9dd610)


<br><b>Задание: Реализовать прямое соединение двух или более таблиц.</b>
<br><b>В данном запросе выводится информация только по тем аэропортам, которые участвуют в полётах:</b>
```
select	F.flight_no,
		F.departure_airport,
		D.airport_name,
		D.city,		
		F.arrival_airport,
		A.airport_name,
		A.city 
from bookings.flights F
  join bookings.airports_data A on F.arrival_airport=A.airport_code
  join bookings.airports_data D on F.departure_airport=D.airport_code;
```
<br>![02_1](https://github.com/user-attachments/assets/dcad70ba-1493-4d9f-9a4b-3b3c2733548c)



<br><b>Задание: Реализовать левостороннее (или правостороннее) соединение двух или более таблиц.</b>
<br><b>В данном запросе выводится информация только по тем полётах, на которые нет билетов:</b>
```
select F.* 
from bookings.flights F
  left join bookings.ticket_flights T on F.flight_id=T.flight_id
where T.flight_id is null;
```
<br>![02_2](https://github.com/user-attachments/assets/7ca09fda-35d2-42cb-a5d6-76a65bec1900)


<br><b>Задание: Реализовать кросс соединение двух или более таблиц.</b>
<br><b>В данном запросе выводится полная информация из двух таблиц, т.е. объединение первой строки первой таблицы с каждой строкой второй таблицы:</b>
```
select * 
from bookings.aircrafts_data
  cross join bookings.airports_data;
```
<br>![02_3](https://github.com/user-attachments/assets/76b0211d-84a6-4772-942d-7f8656756e80)


<br><b>Задание: Реализовать полное соединение двух или более таблиц.</b>
<br><b>Данный запрос выведет все совпадающие элементы, а также все несовпадающие элементы как из одной таблицы, так и из другой:</b>
```
select * 
from bookings.flights F
  full join bookings.aircrafts_data A on F.aircraft_code=A.aircraft_code;
```
<br>![02_4](https://github.com/user-attachments/assets/abf9b31e-4f44-4cbe-b9ea-fba9505c2089)


<br><b>Задание: Реализовать полное соединение двух или более таблиц.</b>
<br><b>Реализован итоговый запрос по таблицам из предметной области полёты. Запрос отображает дынне о всех полётах с указанием доступных мест в самолёте (qnt_seats) и о количестве пассажиров (qnt_boarding_passes):</b>
```
select	F.flight_id,
    	F.flight_no,
    	F.status,
    	D.airport_name as departure_airport_name, 
    	A.airport_name as arrival_airport_name,
    	F.scheduled_departure,
    	timezone(A.timezone, F.scheduled_arrival) as scheduled_arrival_local,
    	F.scheduled_arrival - F.scheduled_departure as scheduled_duration,
    	S.qnt_seats,
    	T.qnt_boarding_passes,
    	F.aircraft_code,
    	C.model
from bookings.flights F
  cross join lateral (select count(*) as qnt_seats from bookings.seats where aircraft_code=F.aircraft_code) S
  cross join lateral (select count(*) as qnt_boarding_passes from bookings.boarding_passes where flight_id=F.flight_id) T
  left join bookings.airports A on F.arrival_airport = A.airport_code
  left join bookings.airports D on F.departure_airport = D.airport_code
  left join bookings.aircrafts_data C on F.aircraft_code = C.aircraft_code
order by T.qnt_boarding_passes desc;
```
<br>![02_5](https://github.com/user-attachments/assets/07e1f4ff-e106-412f-8b24-a9f3467bceff)
