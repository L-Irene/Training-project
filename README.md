## Домашнее задание.

Разработка проекта базы данных интернет магазина для дипломного проекта.

Цель: Спроектировать свою первую базу данных на курсе.

Описание/Пошаговая инструкция выполнения домашнего задания: Реализуйте сущности продукты, категории продуктов, цены, поставщики, производители, покупатели и покупки.
Свои решения для этой схемы приветствуются.

В проекте должны быть:
- схема
- описание таблиц и полей
- примеры бизнес-задач которые решает база

### Схема
![Схема](https://github.com/user-attachments/assets/17bd5c2f-458d-4735-955a-6bb4cbfeffb5)

### Описание таблиц и полей
- Таблица *customer* - Клиент
  - *id* - Уникальный идентификатор клиента
  - *surname* - Фамилия клиента
  - *name* - Имя клиента
  - *patronymic* - Отчество клиента
  - *date_of_birth* - Дата рождения клиента
  - *phone* - Телефон клиента
  - *email* - Электронная почта клиента
  - *address* - Адрес клиента
  - *driver_license_number* - Номер водительского удостоверения клиента
  - *customer_type_id* - Идентификатор типа клиента (ссылка на таблицу customer_type)

- Таблица *employees* - Сотрудник
  - *id* - Уникальный идентификатор сотрудника
  - *surname* - Фамилия сотрудника
  - *name* - Имя сотрудника
  - *patronymic* - Отчество сотрудника
  - *position* - Должность сотрудника
  - *phone_number* - Телефон сотрудника
  - *email* - Электронная почта сотрудника

- Таблица *model* - Модель транспортного средства
  - *id* - Уникальный идентификатор модели
  - *transport_type_id* - Идентификатор типа ТС (ссылка на таблицу transport_type)
  - *brand* - Марка автомобиля
  - *year_of_production* - Год производства
  - *body_type* - Тип кузова
  - *engine_capacity* - Объем двигателя

- Таблица *vehicle* - Транспортное средство
  - *id* - Уникальный идентификатор ТС
  - *model_id* - Идентификатор модели (ссылка на таблицу model)
  - *state* - Статус доступности ТС (ссылка на таблицу reservation_status)
  - *color* - Цвет ТС
  - *license_plate* - Государственный номер ТС

- Таблица *reservation_status* - Справочник статусов брони
  - *id* - Уникальный идентификатор статуса бронирования
  - *name* - Название статуса бронирования

- Таблица *reservation* - Бронирования/заявки на бронь
  - *id* - Уникальный идентификатор бронирования
  - *customer_id* - Идентификатор клиента (ссылка на таблицу customer)
  - *start_date* - Дата начала аренды
  - *finish_date* - Дата окончания аренды
  - *total_cost* - Общая стоимость аренды
  - *employees_id* - Идентификатор сотрудника (ссылка на таблицу employees)
  - *state* - Статус брони (ссылка на таблицу reservation_status)
  - *datetime* - Дата и время создания брони

- Таблица *rental_prices* - Стоимость аренды ТС от типа клиента
  - *id* - Уникальный идентификатор цены аренды
  - *vehicle_id* - Идентификатор автомобиля (ссылка на таблицу vehicle)
  - *customer_type_id* - Идентификатор типа клиента (ссылка на таблицу customer_type)
  - *daily_price* - Ежедневная стоимость аренды

- Таблица *contract* - Договор
  - *id* - Уникальный идентификатор договора
  - *number* - Номер договора
  - *customer_id* - Идентификатор клиента (ссылка на таблицу customer)
  - *employees_id* - Идентификатор сотрудника (ссылка на таблицу employees)
  - *reservation_id* - Идентификатор бронирования (ссылка на аблицу reservation)
  - *payment_status* - Статус оплаты
  - *date* - Дата договора
  - *created_at* - Дата создания договора в системе
  - *updated_at* - Дата последнего обновления договора

- Таблица *reservation_request* - Заявка на бронь 
  - *id* - Уникальный идентификатор запроса на бронирование
  - *reservation_id* - Идентификатор бронирования (ссылка на reservation)
  - *vehicle_id* - Идентификатор автомобиля (ссылка на vehicle)

- Таблица *transport_type* - Тип ТС
  - *id* - Уникальный идентификатор типа ТС
  - *name* - Название типа ТС
  - *category* - Категория типа ТС

- Таблица *customer_type* - Тип клиента (группа)
  - *id* - Уникальный идентификатор типа клиента
  - *name* - Название типа клиента

### Скрипт генерации объектов
---
```SQL
CREATE TABLE "vehicle" (
	"id" serial NOT NULL,
	"transport_type_id" bigint NOT NULL,
	"brand" varchar(100) NOT NULL,
	"model" varchar(100) NOT NULL,
	"year_of_production" bigint NOT NULL,
	"body_type" varchar(32) NOT NULL,
	"engine_capacity" varchar(32) NOT NULL,
	"color" varchar(32) NOT NULL,
	"license_plate" varchar(12) NOT NULL,
	"state" varchar(64) NOT NULL,
	PRIMARY KEY ("id")
);

CREATE TABLE "transport_type" (
	"id" serial NOT NULL UNIQUE,
	"name" varchar(32) NOT NULL,
	PRIMARY KEY ("id")
);

CREATE TABLE "customer" (
	"id" serial NOT NULL UNIQUE,
	"surname" varchar(64) NOT NULL,
	"name" varchar(64) NOT NULL,
	"patronymic" varchar(64),
	"date_of_birth" date NOT NULL,
	"phone" varchar(32) NOT NULL,
	"email" varchar(100) NOT NULL,
	"address" varchar(255) NOT NULL,
	"driver_license_number" varchar(64) NOT NULL,
	"customer_type_id" bigint,
	PRIMARY KEY ("id")
);

CREATE TABLE "rental" (
	"id" serial NOT NULL UNIQUE,
	"vehicle_id" bigint NOT NULL,
	"customer_id" bigint NOT NULL,
	"start_date" date NOT NULL,
	"finish_date" date NOT NULL,
	"total_cost" numeric(10,0) NOT NULL,
	"employee_id" bigint NOT NULL,
	PRIMARY KEY ("id")
);

CREATE TABLE "employee" (
	"id" serial NOT NULL UNIQUE,
	"surname" varchar(64) NOT NULL,
	"name" varchar(64) NOT NULL,
	"patronymic" varchar(64),
	"position" varchar(1024) NOT NULL,
	PRIMARY KEY ("id")
);

CREATE TABLE "rental_prices" (
	"id" serial NOT NULL UNIQUE,
	"vehicle_id" bigint NOT NULL,
	"customer_type_id" bigint NOT NULL,
	"daily_price" numeric(10,0) NOT NULL,
	PRIMARY KEY ("id")
);

CREATE TABLE "customer_type" (
	"id" serial NOT NULL UNIQUE,
	"name" varchar(64) NOT NULL,
	PRIMARY KEY ("id")
);

CREATE TABLE "contract" (
	"id" serial NOT NULL UNIQUE,
	"vehicle_id" bigint NOT NULL,
	"customer_id" bigint NOT NULL,
	"employee_id" bigint NOT NULL,
	"rental_id" bigint NOT NULL,
	"payment_status" varchar(32) NOT NULL,
	"signature_date" date NOT NULL,
	"created_at" timestamp with time zone NOT NULL,
	"updated_at" timestamp with time zone NOT NULL,
	PRIMARY KEY ("id")
);

CREATE TABLE "reservations" (
	"id" serial NOT NULL UNIQUE,
	"vehicle_id" bigint NOT NULL,
	"customer_id" bigint NOT NULL,
	"start_date" date NOT NULL,
	"finish_date" date NOT NULL,
	"reservation_date" timestamp with time zone NOT NULL,
	"state" varchar(64) NOT NULL,
	PRIMARY KEY ("id")
);

ALTER TABLE "vehicle" ADD CONSTRAINT "vehicle_fk1" FOREIGN KEY ("transport_type_id") REFERENCES "transport_type"("id");
ALTER TABLE "customer" ADD CONSTRAINT "customer_fk9" FOREIGN KEY ("customer_type_id") REFERENCES "customer_type"("id");
ALTER TABLE "rental" ADD CONSTRAINT "rental_fk1" FOREIGN KEY ("vehicle_id") REFERENCES "vehicle"("id");
ALTER TABLE "rental" ADD CONSTRAINT "rental_fk2" FOREIGN KEY ("customer_id") REFERENCES "customer"("id");
ALTER TABLE "rental" ADD CONSTRAINT "rental_fk6" FOREIGN KEY ("employee_id") REFERENCES "employee"("id");
ALTER TABLE "rental_prices" ADD CONSTRAINT "rental_prices_fk1" FOREIGN KEY ("vehicle_id") REFERENCES "vehicle"("id");
ALTER TABLE "rental_prices" ADD CONSTRAINT "rental_prices_fk2" FOREIGN KEY ("customer_type_id") REFERENCES "customer_type"("id");
ALTER TABLE "contract" ADD CONSTRAINT "contract_fk1" FOREIGN KEY ("vehicle_id") REFERENCES "vehicle"("id");
ALTER TABLE "contract" ADD CONSTRAINT "contract_fk2" FOREIGN KEY ("customer_id") REFERENCES "customer"("id");
ALTER TABLE "contract" ADD CONSTRAINT "contract_fk3" FOREIGN KEY ("employee_id") REFERENCES "employee"("id");
ALTER TABLE "contract" ADD CONSTRAINT "contract_fk4" FOREIGN KEY ("rental_id") REFERENCES "rental"("id");
ALTER TABLE "reservations" ADD CONSTRAINT "reservations_fk1" FOREIGN KEY ("vehicle_id") REFERENCES "vehicle"("id");
ALTER TABLE "reservations" ADD CONSTRAINT "reservations_fk2" FOREIGN KEY ("customer_id") REFERENCES "customer"("id");
