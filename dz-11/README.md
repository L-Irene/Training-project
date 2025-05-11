## Домашнее задание ##
SQL выборка

### Цель: ###
Научиться джойнить таблицы и использовать условия в SQL выборке


### Описание/Пошаговая инструкция выполнения домашнего задания: ###
Мы подготовили для вас 2 типа задний. Выберите одно из них или выполните оба, если у вас есть желание.

Задание в сфере тестирования. Для этого домашнего занятия вам понадобятся две пустые таблицы:

```sql
CREATE TABLE categories IF NOT EXISTS (
    category_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(32) NOT NULL
);

CREATE TABLE products IF NOT EXISTS (
    product_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(32) NOT NULL,
    category_id VARCHAR(32) REFERENCES categories (category_id),
    price INT,
    rating INT,
    status VARCHAR(32) NOT NULL -- "В наличии" или "Распродан"
);
```
1. Проанализируйте и скорректируйте типы данных в полях этих таблиц, чтобы исключить вставку невалидных с точки зрения бизнес-логики данных.

2. Напишите хранимую процедуру (или скрипт на любом языке программирования) со следующей функциональностью:
сгенерировать 20 категорий
после каждой вставки в таблицу категорий получить текущий category_id с помощью LAST_INSERT_ID() и сгенерировать 10000 товаров в текущей категории
ВАЖНО: все цены товаров должны быть уникальными
суммарно должно получиться 200 тысяч товаров (по 10 тысяч в каждой категории)

3. Напишите запрос, который выведет товары в следующем порядке:

сначала все товары в наличии, отсортированные по возрастанию цены
затем все распроданные товары, тоже отсортированные по возрастанию цены

Ваш запрос должен поддерживать постраничную выдачу (по 50 товаров на страницу). Продумайте наиболее эффективный способ организации такой выдачи с учётом особенностей LIMIT в MySQL. Обратите внимание, что все цены в таблице products уникальные, это поможет вам оптимизировать запрос.

-----------------------------------------------------

### 1. Типы данных ###
1.1. В таблице "products" логически некорректно указан тип данных "VARCHAR(32)" для поля "category_id". Тип данных должен быть числовым.
В таблице "categories" поле "category_id" имеет тип "BIGINT UNSIGNED", поэтому "category_id" в таблице "products" также необходимо скорректировать на "BIGINT UNSIGNED".
```sql
category_id BIGINT UNSIGNED NOT NULL
```
1.2. Для уникальности цен необходимо добавить ограничение ключа для поля "price" 
```sql
price INT UNIQUE
```

### 2. Хранимая процедура ###
``` sql
 --изменение обозначения конца запроса, во избежание ошибок с типовым символом ";".
DELIMITER //  

CREATE PROCEDURE generate_data()
BEGIN
    DECLARE с_counter INT DEFAULT 1;
    DECLARE p_counter INT;
    DECLARE current_category_id BIGINT UNSIGNED;
    DECLARE price INT;
    DECLARE title VARCHAR(32);

-- Для понимания переменных введены следующие их имена: c_counter = category_counter (для ослеживания текущей категории) и p_counter = product_counter (для отслеживания текущего товара).

    -- Генерация категорий
    WHILE с_counter <= 20 DO
        SET title = CONCAT('Category ', с_counter);
        INSERT INTO categories (title) VALUES (title);
        SET current_category_id = LAST_INSERT_ID();

-- "WHILE с_counter <= 20" выполняется 20 раз для создания 20 категорий.
-- После вставки категории в таблицу "categories" получаем соответствующий идентификатор для категории "current_category_id" с помощью LAST_INSERT_ID().

        -- Генерация товаров
        SET p_counter = 1;
        WHILE p_counter <= 10000 DO
            SET price = (с_counter - 1) * 100000 + p_counter; -- Расчет и генерация уникальной цены для каждого товара в категории. 
            SET title = CONCAT('Product ', p_counter, ' in Category ', с_counter);
            INSERT INTO products (title, category_id, price, rating, status)
            VALUES (title, current_category_id, price, FLOOR(1 + (RAND() * 5)), IF(p_counter % 2 = 0, 'В наличии', 'Распродан'));
            SET p_counter = p_counter + 1;
        END WHILE;

        SET с_counter = с_counter + 1;
    END WHILE;
END //

-- По формуле "(с_counter - 1) * 100000 + p_counter" рассчитывается уникальная цена товара, где с_counter — номер категории, а p_counter — номер товара в категории. 
-- Рейтинг товара генерируется случайным образом от 1 до 5.
-- Статус товара генерируется случайным образом: "В наличии" или "Распродан".

DELIMITER ;
```

### 3. Запрос на вывод товара ###
```sql
SET @page = 1; -- номер страницы
SET @page_size = 50; -- количество товаров на странице

SELECT * FROM (
    SELECT product_id, title, category_id, price, rating, status
    FROM products
    WHERE status = 'В наличии'
    ORDER BY price ASC
    UNION ALL
    SELECT product_id, title, category_id, price, rating, status
    FROM products
    WHERE status = 'Распродан'
    ORDER BY price ASC
) AS combined_products
LIMIT (@page - 1) * @page_size, @page_size;
```
