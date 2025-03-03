### Описание/Пошаговая инструкция выполнения домашнего задания: ###

Используя операторы DDL создайте на примере схемы интернет-магазина:
- Базу данных.
- Табличные пространства и роли.
- Схему данных.
- Таблицы своего проекта, распределив их по схемам и табличным пространствам.

## *SQL скрипт* ##

  ```sql

CREATE DATABASE rental_service
WITH
OWNER = postgres   
ENCODING = 'UTF8'    
TABLESPACE = pg_default  
CONNECTION LIMIT = -1;  

CREATE TABLESPACE current_data LOCATION '/pul/pg_sql/current_d';   
CREATE TABLESPACE archive_data LOCATION '/pul/pg_sql/archive_d';

CREATE SCHEMA current AUTHORIZATION admin;   
CREATE SCHEMA archive AUTHORIZATION admin;

CREATE ROLE admin WITH   
LOGIN   
SUPERUSER   
CREATEDB   
CREATEROLE   
INHERIT   
NOREPLICATION   
CONNECTION LIMIT 5   
PASSWORD 'password';   

CREATE ROLE moderator_readonly WITH   
LOGIN   
NOSUPERUSER
NOCREATEDB   
NOCREATEROLE   
INHERIT   
NOREPLICATION   
CONNECTION LIMIT -1   
PASSWORD 'password';   

GRANT CONNECT ON DATABASE rental_service TO moderator_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA current TO moderator_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA current GRANT SELECT ON TABLES TO moderator_readonly;   

CREATE TABLE current.vehicle (
	"id" serial NOT NULL,
	"model_id" bigint NOT NULL,
	"color" varchar(32) NOT NULL,
	"license_plate" varchar(12) NOT NULL UNIQUE,
	"status_id" bigint NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.model (
	"id" serial NOT NULL,
	"transport_type_id" bigint NOT NULL,
	"brand" varchar(100) NOT NULL,
	"year_of_production" bigint NOT NULL,
	"body_type" varchar(32) NOT NULL,
	"engine_capacity" varchar(32) NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.transport_type (
	"id" serial NOT NULL UNIQUE,
	"name" varchar(32) NOT NULL,
	"category" varchar(32) NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.customer (
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
) TABLESPACE current_data;

CREATE TABLE current.rental (
	"id" serial NOT NULL UNIQUE,
	"vehicle_id" bigint NOT NULL,
	"customer_id" bigint NOT NULL,
	"start_date" date NOT NULL,
	"finish_date" date NOT NULL,
	"total_cost" numeric(10,0) NOT NULL,
	"employees_id" bigint NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.employees (
	"id" serial NOT NULL UNIQUE,
	"surname" varchar(64) NOT NULL,
	"name" varchar(64) NOT NULL,
	"patronymic" varchar(64),
	"position_id" bigint NOT NULL,
	"phone_number" varchar(32) NOT NULL,
	"email" varchar(100) NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.rental_prices (
	"id" serial NOT NULL UNIQUE,
	"vehicle_id" bigint NOT NULL,
	"customer_type_id" bigint NOT NULL,
	"daily_price" numeric(10,0) NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.customer_type (
	"id" serial NOT NULL UNIQUE,
	"name" varchar(64) NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.contract (
	"id" serial NOT NULL UNIQUE,
	"number" varchar(32) NOT NULL,
	"vehicle_id" bigint NOT NULL,
	"customer_id" bigint NOT NULL,
	"employees_id" bigint NOT NULL,
	"rental_id" bigint NOT NULL,
	"payment_status" varchar(32) NOT NULL,
	"signature_date" date NOT NULL,
	"created_at" timestamp with time zone NOT NULL,
	"updated_at" timestamp with time zone NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.reservation (
	"id" serial NOT NULL UNIQUE,
	"vehicle_id" bigint NOT NULL,
	"customer_id" bigint NOT NULL,
	"start_date" date NOT NULL,
	"finish_date" date NOT NULL,
	"reservation_date" timestamp with time zone NOT NULL,
	"state" bigint NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.payment (
   	 "id" serial NOT NULL UNIQUE,
    	 "contract_id" bigint NOT NULL,
     	 "amount" numeric(10,0) NOT NULL,
   	 "payment_date" date NOT NULL,
   	 "payment_method" varchar(64) NOT NULL,
   	 PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.employee_position (
  	  "id" serial NOT NULL UNIQUE,
   	 "name" varchar(64) NOT NULL,
   	 PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.vehicle_status (
   	 "id" serial NOT NULL UNIQUE,
   	 "name" varchar(64) NOT NULL,
   	 PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE current.reservation_status (
   	 "id" serial NOT NULL UNIQUE,
  	  "name" varchar(64) NOT NULL,
   	 PRIMARY KEY ("id")
) TABLESPACE current_data;

CREATE TABLE archive.vehicle (
	"id" serial NOT NULL,
	"model_id" bigint NOT NULL,
	"color" varchar(32) NOT NULL,
	"license_plate" varchar(12) NOT NULL UNIQUE,
	"status_id" bigint NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE archive_data;

CREATE TABLE archive.rental (
	"id" serial NOT NULL UNIQUE,
	"vehicle_id" bigint NOT NULL,
	"customer_id" bigint NOT NULL,
	"start_date" date NOT NULL,
	"finish_date" date NOT NULL,
	"total_cost" numeric(10,0) NOT NULL,
	"employees_id" bigint NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE archive_data;

CREATE TABLE archive.reservation (
	"id" serial NOT NULL UNIQUE,
	"vehicle_id" bigint NOT NULL,
	"customer_id" bigint NOT NULL,
	"start_date" date NOT NULL,
	"finish_date" date NOT NULL,
	"reservation_date" timestamp with time zone NOT NULL,
	"state" bigint NOT NULL,
	PRIMARY KEY ("id")
) TABLESPACE archive_data;

ALTER TABLE current.model ADD CONSTRAINT "model_fk1" FOREIGN KEY ("transport_type_id") REFERENCES current.transport_type("id");
ALTER TABLE current.customer ADD CONSTRAINT "customer_fk9" FOREIGN KEY ("customer_type_id") REFERENCES current.customer_type("id");
ALTER TABLE current.rental ADD CONSTRAINT "rental_fk1" FOREIGN KEY ("vehicle_id") REFERENCES current.vehicle("id");
ALTER TABLE current.rental ADD CONSTRAINT "rental_fk2" FOREIGN KEY ("customer_id") REFERENCES current.customer("id");
ALTER TABLE current.rental ADD CONSTRAINT "rental_fk6" FOREIGN KEY ("employees_id") REFERENCES current.employees("id");
ALTER TABLE current.rental_prices ADD CONSTRAINT "rental_prices_fk1" FOREIGN KEY ("vehicle_id") REFERENCES current.vehicle("id");
ALTER TABLE current.rental_prices ADD CONSTRAINT "rental_prices_fk2" FOREIGN KEY ("customer_type_id") REFERENCES current.customer_type("id");
ALTER TABLE current.contract ADD CONSTRAINT "contract_fk1" FOREIGN KEY ("vehicle_id") REFERENCES current.vehicle("id");
ALTER TABLE current.contract ADD CONSTRAINT "contract_fk2" FOREIGN KEY ("customer_id") REFERENCES current.customer("id");
ALTER TABLE current.contract ADD CONSTRAINT "contract_fk3" FOREIGN KEY ("employees_id") REFERENCES current.employees("id");
ALTER TABLE current.contract ADD CONSTRAINT "contract_fk4" FOREIGN KEY ("rental_id") REFERENCES current.rental("id");
ALTER TABLE current.reservation ADD CONSTRAINT "reservation_fk1" FOREIGN KEY ("vehicle_id") REFERENCES current.vehicle("id");
ALTER TABLE current.reservation ADD CONSTRAINT "reservation_fk2" FOREIGN KEY ("customer_id") REFERENCES current.customer("id");
ALTER TABLE current.payment ADD CONSTRAINT "payment_fk1" FOREIGN KEY ("contract_id") REFERENCES current.contract("id");
ALTER TABLE current.employees ADD CONSTRAINT "employees_fk1" FOREIGN KEY ("position_id") REFERENCES current.employee_position("id");
ALTER TABLE current.vehicle ADD CONSTRAINT "vehicle_fk2" FOREIGN KEY ("status_id") REFERENCES current.vehicle_status("id");
ALTER TABLE current.vehicle ADD CONSTRAINT "vehicle_fk3" FOREIGN KEY ("model_id") REFERENCES current.model("id");
ALTER TABLE current.reservation ADD CONSTRAINT "reservation_fk3" FOREIGN KEY ("state") REFERENCES current.reservation_status("id");

CREATE INDEX idx_customer_type_id ON current.customer (customer_type_id);
CREATE INDEX idx_reservation_dates ON current.reservation (start_date, finish_date);
CREATE INDEX idx_employees_id ON current.employees (id);
CREATE INDEX idx_rental_prices_daily_price ON current.rental_prices (daily_price);
CREATE INDEX idx_payment_date ON current.payment (payment_date);
CREATE INDEX idx_vehicle_model_id ON current.vehicle (model_id);
CREATE INDEX idx_vehicle_status_id ON current.vehicle (status_id);
CREATE INDEX idx_model_transport_type_id ON current.model (transport_type_id);

```
