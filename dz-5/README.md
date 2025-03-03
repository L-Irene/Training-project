## Домашнее задание ##

### Цель: ###
1. Написать запрос с конструкциями SELECT, JOIN;
2. Написать запрос с добавлением данных INSERT INTO;   
3. Написать запрос с обновлением данных с UPDATE FROM;   
4. Использовать using для оператора DELETE.   

### Описание/Пошаговая инструкция выполнения домашнего задания: ###
- Напишите запрос по своей базе с регулярным выражением, добавьте пояснение, что вы хотите найти.
- Напишите запрос по своей базе с использованием LEFT JOIN и INNER JOIN, как порядок соединений в FROM влияет на результат? Почему?
- Напишите запрос на добавление данных с выводом информации о добавленных строках.
- Напишите запрос с обновлением данные используя UPDATE FROM.
- Напишите запрос для удаления данных с оператором DELETE используя join с другой таблицей с помощью using.


 -- Напишите запрос по своей базе с регулярным выражением, добавьте пояснение, что вы хотите найти.   
 Найдём все регистрационные номера ТС, в которых содержится любые три из представленных букв с точным совпадением регистра, а также три цифры в части номера региона.
```sql
SELECT *  
FROM current.vehicle v
WHERE v.license_plate ~ '^[АВЕОТ]{3}\d{3}';
```

 -- Напишите запрос по своей базе с использованием LEFT JOIN и INNER JOIN, как порядок соединений в FROM влияет на результат? Почему?
 С помощью запроса возвращаем все записи из таблицы "employees" (Сотрудники), где есть соответствующие записи в таблице "employee_position" (Справочник должностей сотрудников). Фактически, возвращает только те записи, у которых есть совпадения в обеих таблицах.
```sql
SELECT current.employees.id AS employee_id, 
       current.employees.name AS employee_name, 
       current.employees.surname AS employee_surname, 
       current.employee_position.name AS position_name
FROM current.employees
INNER JOIN current.employee_position 
ON current.employees.position_id = current.employee_position.id;
```
 С помощью запроса возвращаем все записи из таблицы "employees" (Сотрудники), где есть соответствующие записи из таблицы "employee_position" (Справочник должностей сотрудников). Фактически, возвращает все записи из левой таблицы - "employees" и соответствующие записи из правой таблицы - "employee_position", если есть совпадения; если совпадение не найдено - для правой таблицы поле значения будет содержать "NULL".
```sql
SELECT current.employees.id AS employee_id, 
       current.employees.name AS employee_name, 
       current.employees.surname AS employee_surname, 
       current.employee_position.name AS position_name
FROM current.employees
LEFT JOIN current.employee_position 
ON current.employees.position_id = current.employee_position.id;
```

 -- Напишите запрос на добавление данных с выводом информации о добавленных строках.
 С помощью запроса добавлен новый клиент в одноимённую таблицу (customer), запрос донасыщен требованием о выводе информации о добавленной строке.
```sql
INSERT INTO current.customer (surname, name, date_of_birth, phone, email, address, driver_license_number)
VALUES ('Иванов', 'Иван', '1985-10-24', '+79165432117', 'ivanI@yandex.com', 'ул. Дмитриева, д.5', '9945 333888')
RETURNING id, name, surname, driver_license_number;
```

 -- Напишите запрос с обновлением данные используя UPDATE FROM.
 В таблице "vehicle" (Транспортное средство) обновим идентификатор статуса доступности ТС, установив значение идентификатора равное "2" для тех ТС, которые есть в таблице "rental" (Аренда транспортного средства), соответственно, были арендованы и дата окончания аренды (finish_date) которых наступила ранее 12 января 2025 года.
```sql
UPDATE current.vehicle v
SET status_id = 2
FROM current.rental r
WHERE v.id = r.vehicle_id AND r.finish_date < '12-01-2025';
```

 -- Напишите запрос для удаления данных с оператором DELETE используя join с другой таблицей с помощью using.
С помощью текущего запроса из таблицы "reservation" (Заявка на бронирование ТС) будут удалены те записи, где есть соответствующие им записи в таблице "rental" (Аренда транспортного средства) с уже завершённой датой аренды (в сравнении с текущей).  
```sql
DELETE FROM current.reservation r
USING current.rental re
WHERE r.vehicle_id = re.vehicle_id AND r.customer_id = re.customer_id AND re.finish_date < current_date;
```
