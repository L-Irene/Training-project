## Домашнее задание ##
Добавляем в базу хранимые процедуры и триггеры

### Цель: ###
Создавать пользователей, процедуры и триггеры.

### Описание/Пошаговая инструкция выполнения домашнего задания: ###
1. Создать пользователей client, manager.
2. Создать процедуру выборки товаров с использованием различных фильтров: категория, цена, производитель, различные дополнительные параметры.

   2.1. Также в качестве параметров передавать по какому полю сортировать выборку, и параметры постраничной выдачи
   
   2.2. дать права да запуск процедуры пользователю client
3. Создать процедуру get_orders - которая позволяет просматривать отчет по продажам за определенный период (час, день, неделя) с различными уровнями группировки (по товару, по категории, по производителю).
   
   3.1. Права дать пользователю manager

---------------------------------------------------------
1. Создание пользователей
```sql
CREATE USER 'client'@'localhost' IDENTIFIED BY 'cpassword';
CREATE USER 'manager'@'localhost' IDENTIFIED BY 'mpassword';
```
2. Процедура выборки товаров
```sql
DELIMITER //

CREATE PROCEDURE get_products(
    IN category_filter VARCHAR(160),
    IN price_min_filter DECIMAL(10, 2),
    IN price_max_filter DECIMAL(10, 2),
    IN manufacturer_filter VARCHAR(255),
    IN additional_params_filter TEXT,
    IN sort_field VARCHAR(160),
    IN sort_order VARCHAR(10),
    IN page_number INT,
    IN page_size INT
)
BEGIN
    SELECT *
    FROM products
    WHERE (category_filter IS NULL OR category = category_filter)
      AND (price_min_filter IS NULL OR price >= price_min_filter)
      AND (price_max_filter IS NULL OR price <= price_max_filter)
      AND (manufacturer_filter IS NULL OR manufacturer = manufacturer_filter)
      AND (additional_params_filter IS NULL OR additional_params LIKE CONCAT('%', additional_params_filter, '%'))
    ORDER BY 
        CASE 
            WHEN sort_field = 'category' AND sort_order = 'ASC' THEN category 
            WHEN sort_field = 'category' AND sort_order = 'DESC' THEN category DESC 
            WHEN sort_field = 'price' AND sort_order = 'ASC' THEN price 
            WHEN sort_field = 'price' AND sort_order = 'DESC' THEN price DESC 
            WHEN sort_field = 'manufacturer' AND sort_order = 'ASC' THEN manufacturer 
            WHEN sort_field = 'manufacturer' AND sort_order = 'DESC' THEN manufacturer DESC 
            ELSE product_id
        END
    LIMIT (page_number - 1) * page_size, page_size;
END //

DELIMITER ;
```
2.2. Права для пользователя client
```sql
GRANT EXECUTE ON PROCEDURE get_products TO 'client'@'localhost';
```
3. Процедура get_orders - которая позволяет просматривать отчет по продажам
```sql
DELIMITER //

CREATE PROCEDURE get_orders(
    IN start_date DATETIME,
    IN end_date DATETIME,
    IN group_by_field VARCHAR(50)
)
BEGIN
    SELECT 
        CASE 
            WHEN group_by_field = 'product' THEN product_id
            WHEN group_by_field = 'category' THEN category_id
            WHEN group_by_field = 'manufacturer' THEN manufacturer_id
            ELSE order_id
        END AS group_id,
        CASE 
            WHEN group_by_field = 'product' THEN product_name
            WHEN group_by_field = 'category' THEN category_name
            WHEN group_by_field = 'manufacturer' THEN manufacturer_name
            ELSE 'All Orders'
        END AS group_name,
        SUM(quantity) AS total_quantity,
        SUM(total_price) AS total_sales
    FROM orders
    WHERE order_date BETWEEN start_date AND end_date
    GROUP BY 
        CASE 
            WHEN group_by_field = 'product' THEN product_id
            WHEN group_by_field = 'category' THEN category_id
            WHEN group_by_field = 'manufacturer' THEN manufacturer_id
            ELSE order_id
        END
    ORDER BY group_id;
END //

DELIMITER ;
```
3.1 Права для пользователя manager
```sql
GRANT EXECUTE ON PROCEDURE get_orders TO 'manager'@'localhost';
```
