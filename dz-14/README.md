## Домашнее задание ##
Индексы

### Цель: ###
Использовать индексы в MySQL.


### Описание/Пошаговая инструкция выполнения домашнего задания: ###
Пересматриваем индексы на своем проекте. По необходимости меняем.

Задача - сделать полнотекстовый индекс, который ищет по свойствам, названию товара и описанию. Если нет аналогичной задачи в проекте - имитируем.

Итог: анализируем свой проект - добавляем или обновляем индексы.

в README пропишите какие индексы были изменены или добавлены.

explain и результаты выборки без индекса и с индексом.

Реализация полнотекстового индекса.

-------------------------------------------------
Полнотекстовый индекс будет создан на основе существующей таблицы "customer" (клиент). 
#### Таблица и ранее созданные индексы ####
```sql
CREATE TABLE "customer" (
	"id" serial NOT NULL UNIQUE,
	"surname" varchar(64) NOT NULL,
	"name" varchar(64) NOT NULL,
	"patronymic" varchar(64),
	"date_of_birth" date NOT NULL,
	"phone" varchar(32) NOT NULL,
	"email" varchar(100) NOT NULL,
	"address" varchar(255) NOT NULL,
	"driver_license_number" varchar(64) NOT NULL UNIQUE,
	"customer_type_id" bigint,
	PRIMARY KEY ("id")
);

CREATE INDEX idx_customer_type_id ON customer (customer_type_id);
CREATE INDEX idx_customer_phone ON customer (phone);
CREATE INDEX idx_customer_email ON customer (email);
```

- Таблица *customer* - Клиент
  - *id* - Уникальный идентификатор клиента;
  - *surname* - Фамилия клиента;
  - *name* - Имя клиента;
  - *patronymic* - Отчество клиента;
  - *date_of_birth* - Дата рождения клиента;
  - *phone* - Телефон клиента;
  - *email* - Электронная почта клиента;
  - *address* - Адрес клиента;
  - *driver_license_number* - Номер водительского удостоверения клиента;
  - *customer_type_id* - Идентификатор типа клиента (ссылка на таблицу customer_type).


1. Добавим текстовое поле "description" в таблицу "customer" для описания характеристики клиента.
```sql
ALTER TABLE customer ADD COLUMN description TEXT;
```
2. Добавляем записи с описанием клиентов.
```sql
UPDATE customer SET description = 'Опытный водитель с большим безаварийным стажем' WHERE id = 1;
UPDATE customer SET description = 'Новичок, метрики стиля вождения - спокойный, аккуратный, аварий по документам не зафиксировано' WHERE id = 2;
UPDATE customer SET description = 'Имеет профессиональный опыт работы с автотранспортом и спецтехникой' WHERE id = 4;
UPDATE customer SET description = 'Внимательный и аккуратный' WHERE id = 5;
```
3. Полнотекстовый индекс на соответствующее добавленное поле.
```sql
ALTER TABLE customer ADD FULLTEXT(description);
```
4. Проверка запросов.

Без использования нового индекса:
```sql
EXPLAIN SELECT * FROM customer WHERE description LIKE '%опытный%';
```
С использованием нового индекса:
```sql
EXPLAIN SELECT * FROM customer WHERE MATCH(description) AGAINST('опытный');
```

*Набора тестовых данных недостаточно для фиксации изменений работы системы и поиска. Однако, обычно, запрос с полнотекстовым индексом выполняется быстрее.
