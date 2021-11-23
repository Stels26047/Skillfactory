/*Задание 4.1 База данных содержит список аэропортов практически всех крупных городов России.
В большинстве городов есть только один аэропорт. Исключение составляет:*/
SELECT a.city,
       count(a.airport_name)
FROM dst_project.airports a
GROUP BY a.city
HAVING count(a.airport_name) > 1
--Задание 4.2  
/*Вопрос 1. Таблица рейсов содержит всю информацию о прошлых, текущих и запланированных рейсах.
Сколько всего статусов для рейсов определено в таблице?*/
SELECT count(DISTINCT status)
FROM dst_project.flights
/*Вопрос 2. Какое количество самолетов находятся в воздухе на момент среза в базе (статус рейса «самолёт уже вылетел и находится в воздухе»).*/
SELECT count(flight_id)
FROM dst_project.flights
WHERE status = 'Departed'
/*Вопрос 3. Места определяют схему салона каждой модели. Сколько мест имеет самолет модели  (Boeing 777-300)?*/
SELECT a.model,
       count(s.seat_no)
FROM dst_project.aircrafts a
LEFT JOIN dst_project.seats s ON a.aircraft_code = s.aircraft_code
WHERE a.model = 'Boeing 777-300'
GROUP BY a.model
/*Вопрос 4. Сколько состоявшихся (фактических) рейсов было совершено между 1 апреля 2017 года и 1 сентября 2017 года?*/
SELECT count(f.flight_id)
FROM dst_project.flights f
WHERE f.status = 'Arrived'
  AND f.actual_arrival BETWEEN '2017-04-01' AND '2017-09-01'
--Задание 4.3
/*Вопрос 1. Сколько всего рейсов было отменено по данным базы?*/
SELECT count(f.flight_id)
FROM dst_project.flights f
WHERE f.status = 'Cancelled'
/*Вопрос 2. Сколько самолетов моделей типа Boeing, Sukhoi Superjet, Airbus находится в базе авиаперевозок?*/ --Boeing:

SELECT count(a.model)
FROM dst_project.aircrafts a
WHERE a.model like 'Boeing%'
--Sukhoi Superjet:

SELECT count(a.model)
FROM dst_project.aircrafts a
WHERE a.model like 'Sukhoi Superjet%'
--Airbus:

SELECT count(a.model)
FROM dst_project.aircrafts a
WHERE a.model like 'Airbus%'
/*Вопрос 3. В какой части (частях) света находится больше аэропортов?*/
SELECT a.timezone,
       count(a.airport_name)
FROM dst_project.airports a
WHERE a.timezone like 'Europe/%'
GROUP BY a.timezone
UNION ALL
SELECT 'TOTAL Europe',
       count(a.airport_name)
FROM dst_project.airports a
WHERE a.timezone like 'Europe/%'
UNION ALL
SELECT a.timezone,
       count(a.airport_name)
FROM dst_project.airports a
WHERE a.timezone like 'Australia/%'
GROUP BY a.timezone
UNION ALL
SELECT 'TOTAL Australia',
       count(a.airport_name)
FROM dst_project.airports a
WHERE a.timezone like 'Australia/%'
UNION ALL
SELECT a.timezone,
       count(a.airport_name)
FROM dst_project.airports a
WHERE a.timezone like 'Europe/%'
  OR a.timezone like 'Asia/%'
GROUP BY a.timezone
UNION ALL
SELECT 'TOTAL Europe, Asia,',
       count(a.airport_name)
FROM dst_project.airports a
WHERE a.timezone like 'Europe/%'
  OR a.timezone like 'Asia/%'
UNION ALL
SELECT a.timezone,
       count(a.airport_name)
FROM dst_project.airports a
WHERE a.timezone like 'Asia/%'
GROUP BY a.timezone
UNION ALL
SELECT 'TOTAL Asia',
       count(a.airport_name)
FROM dst_project.airports a
WHERE a.timezone like 'Asia/%'
ORDER BY 2 DESC
LIMIT 1
/*Вопрос 4. У какого рейса была самая большая задержка прибытия за все время сбора данных? Введите id рейса (flight_id).*/
SELECT flight_id,
       (actual_arrival - scheduled_arrival) time_delta
FROM dst_project.flights
WHERE actual_arrival IS NOT NULL
  AND scheduled_arrival IS NOT NULL
ORDER BY 2 DESC
LIMIT 1
--Задание 4.4
/*Вопрос 1. Когда был запланирован самый первый вылет, сохраненный в базе данных?*/
SELECT min(actual_departure)
FROM dst_project.flights
WHERE actual_departure IS NOT NULL
/*Вопрос 2. Сколько минут составляет запланированное время полета в самом длительном рейсе?*/
SELECT extract(HOUR
               FROM scheduled_arrival-scheduled_departure)*60 + extract(MINUTE
                                                                        FROM scheduled_arrival-scheduled_departure)
FROM dst_project.flights
WHERE actual_departure IS NOT NULL
  AND actual_arrival IS NOT NULL
ORDER BY 1 DESC
LIMIT 1
/*Вопрос 3. Между какими аэропортами пролегает самый длительный по времени запланированный рейс?*/
SELECT departure_airport,
       arrival_airport,
       (actual_arrival - scheduled_arrival) difference
FROM dst_project.flights
WHERE (actual_arrival IS NOT NULL
       AND scheduled_arrival IS NOT NULL)
  AND (departure_airport='DME'
       AND arrival_airport='UUS')
  OR (departure_airport='DME'
      AND arrival_airport='AAQ')
  OR (departure_airport='DME'
      AND arrival_airport='PCS')
  OR (departure_airport='SVO'
      AND arrival_airport='UUS')
GROUP BY departure_airport,
         arrival_airport,
         difference
ORDER BY 3 DESC
OFFSET 1
LIMIT 1
/*Вопрос 4. Сколько составляет средняя дальность полета среди всех самолетов в минутах? Секунды округляются в меньшую сторону (отбрасываются до минут).*/
SELECT floor(date_part('hour', avg(f.scheduled_arrival - f.scheduled_departure))*60 + date_part('minute', avg(f.scheduled_arrival - f.scheduled_departure)) + date_part('second', avg(f.scheduled_arrival - f.scheduled_departure))/60)
FROM dst_project.flights f
LEFT JOIN dst_project.aircrafts a ON f.aircraft_code = a.aircraft_code
WHERE (actual_arrival IS NOT NULL
       AND scheduled_arrival IS NOT NULL)
--Задание 4.5
/*Вопрос 1. Мест какого класса у SU9 больше всего?*/
SELECT s.fare_conditions,
       count(s.seat_no)
FROM dst_project.aircrafts a
LEFT JOIN dst_project.seats s ON a.aircraft_code = s.aircraft_code
WHERE a.aircraft_code = 'SU9'
GROUP BY s.fare_conditions
ORDER BY 2 DESC
LIMIT 1
/*Вопрос 2. Какую самую минимальную стоимость составило бронирование за всю историю?*/
SELECT min(amount)
FROM dst_project.ticket_flights
/*Вопрос 3. Какой номер места был у пассажира с id = 4313 788533?*/
SELECT seat_no
FROM dst_project.boarding_passes b
LEFT JOIN dst_project.tickets t ON b.ticket_no = t.ticket_no
WHERE t.passenger_id='4313 788533'
--Задание 5.1
/*Вопрос 1. Анапа — курортный город на юге России. Сколько рейсов прибыло в Анапу за 2017 год?('AAQ'-Это Анапа)*/
SELECT f.arrival_airport,
       count(f.arrival_airport)
FROM dst_project.flights f
WHERE f.scheduled_departure BETWEEN '2017-01-01' AND '2017-12-31'
GROUP BY f.arrival_airport
LIMIT 1
/*Вопрос 2. Сколько рейсов из Анапы вылетело зимой 2017 года?*/
SELECT f.departure_airport,
       count(f.departure_airport)
FROM dst_project.flights f
WHERE f.scheduled_departure BETWEEN '2017-01-01' AND '2017-02-28'
GROUP BY f.departure_airport
UNION ALL
SELECT f.departure_airport,
       count(f.departure_airport)
FROM dst_project.flights f
WHERE f.scheduled_departure BETWEEN '2017-12-01' AND '2017-12-31'
GROUP BY f.departure_airport
ORDER BY 1
LIMIT 1
/*Вопрос 3. Посчитайте количество отмененных рейсов из Анапы за все время.)*/
SELECT count(f.status)
FROM dst_project.flights f
WHERE f.arrival_airport ='AAQ'
  AND f.status = 'Cancelled'
GROUP BY f.arrival_airport
/*Вопрос 4. Сколько рейсов из Анапы не летают в Москву?*/
SELECT count(f.flight_id)
FROM dst_project.flights f
WHERE f.departure_airport ='AAQ'
  AND f.arrival_airport !='SVO'
/*Вопрос 5. Какая модель самолета летящего на рейсах из Анапы имеет больше всего мест?*/
SELECT model,
       count(seat_no)
FROM dst_project.aircrafts a
JOIN dst_project.seats s ON a.aircraft_code = s.aircraft_code
GROUP BY model
ORDER BY 2 DESC
LIMIT 1