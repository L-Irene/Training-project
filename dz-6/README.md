### Домашнее задание ###

#### Описание/Пошаговая инструкция выполнения домашнего задания: ####   
Создать индексы на БД, которые ускорят доступ к данным.    
#### Необходимо: ####   
Создать индекс к какой-либо из таблиц вашей БД.   
Прислать текстом результат команды explain, в которой используется данный индекс.   
Реализовать индекс для полнотекстового поиска.    
Реализовать индекс на часть таблицы или индекс на поле с функцией.    
Создать индекс на несколько полей.      
  Написать комментарии к каждому из индексов. Описать что и как делали и с какими проблемами столкнулись    

```sql
CREATE TABLE current.vehicle (
  "id" serial NOT NULL,
  "model_id" bigint NOT NULL,
  "color" varchar(32) NOT NULL,
  "license_plate" varchar(12) NOT NULL UNIQUE,
  "status_id" bigint NOT NULL,
  PRIMARY KEY ("id")
) TABLESPACE current_data;
```
- Создать индекс к какой-либо из таблиц вашей БД.    
```sql
CREATE INDEX idx_vehicle_model_id ON current.vehicle (model_id);    
CREATE INDEX idx_vehicle_status_id ON current.vehicle (status_id);
```
- Прислать текстом результат команды explain, в которой используется данный индекс.     
  Задали поиск по таблице с помощью индекса, где идентификатор модели = 3.
```sql
EXPLAIN SELECT * FROM current.vehicle WHERE model_id = 3;
```
Результат:  
  *Проблема:* В результате - PostgreSQL не использовал индекс, вероятно, из-за небольшого размера таблицы (таблица состоит из 54 строк). 
```sql
Seq Scan on vehicle  (cost=0.00..1.65 rows=17 width=47)
  Filter: (model_id = 3)
```
- Реализовать индекс для полнотекстового поиска.        
```sql
CREATE INDEX idx_vehicle_color_fts ON current.vehicle USING GIN (to_tsvector('russian', color));
```
  Создан новый индекс, с применением оператора USING GIN (для гибкого полнотекстового поиска).    
  to_tsvector('russian', color) выражение преобразует текстовое поле ("color") в структуру данных, с использованием определённой конфигурации языка (russian).     

- Реализовать индекс на часть таблицы или индекс на поле с функцией.  
 Допустим, что status_id = 6, что соответствует фактическому статусу дсотупности ТС "В ремонте" (repair).
 Тогда создадим ограниченный индекс, который должен ускорить поиск по license_plate и искать только среди ТС, имеющих соответствующий статус "В ремонте" (идентификатор статуса = 6). 
```sql
CREATE INDEX idx_vehicle_repair_license_plate ON current.vehicle (license_plate) WHERE status_id = 6;
```

- Создать индекс на несколько полей.     
  Создадим индекс на поля model_id и status_id.
```sql
CREATE INDEX idx_vehicle_model_status ON current.vehicle (model_id, status_id);
```
