## Домашнее задание ##
Оконные функции (задание для QA)

### Цель: ###
научиться проектировать, использовать и анализировать оконные функции с учётом граничных случаев;


### Описание/Пошаговая инструкция выполнения домашнего задания: ###
Для этого домашнего занятия вам понадобятся пустые таблицы:
```sql
CREATE TABLE stores (
    store_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    address VARCHAR(50) NOT NULL
);

CREATE TABLE sales (
    sale_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    store_id BIGINT REFERENCES stores (store_id),
    date TIMESTAMP NOT NULL,
    sale_amount DECIMAL(10,2) NOT NULL
);
```
1. Напишите хранимую процедуру (или скрипт на любом языке программирования) со следующей функциональностью:
- сгенерировать 10 магазинов в таблице stores
- сгенерировать 100000 продаж в таблице sales за последние 2 года
- продажи должны быть распределены НЕРАВНОМЕРНО между магазинами (70-75% продаж должны быть в каком-то одном магазине)
2. Напишите запрос, который выведет нарастающий итог продаж по каждому магазину с группировкой по месяцам.
3. Напишите запрос, который выведет 7-дневное скользящее среднее за последний месяц по самому плодовитому магазину.
4. Опишите, какие граничные случаи вы учли в своих запросах:
- по нарастающему итогу часть информации была на занятии
- по скользящему среднему нужно подумать самостоятельно

-------------------------------------------------------------
1. Хранимая процедура
```sql
DELIMITER //

CREATE PROCEDURE generate_sales_data()
BEGIN
    DECLARE shop_counter INT DEFAULT 1;
    DECLARE trade_counter INT DEFAULT 1;
    DECLARE random_store_id INT;
    DECLARE random_amount DECIMAL(10,2);
    DECLARE random_date TIMESTAMP;
    DECLARE most_productive_store_id INT;
    DECLARE most_productive_store_sales_count INT DEFAULT 0;

-- shop_counter для отслеживания текущего магазина
-- trade_counter для отслеживания продажи текущего магазина

    -- Генерация магазинов
    WHILE shop_counter <= 10 DO
        INSERT INTO stores (address) VALUES (CONCAT('Address ', shop_counter));
        SET shop_counter = shop_counter + 1;
    END WHILE;

    -- Выбор самого плодовитого магазина
    SET most_productive_store_id = FLOOR(1 + (RAND() * 10));

    -- Генерация продаж
    WHILE trade_counter <= 100000 DO
        SET random_store_id = FLOOR(1 + (RAND() * 10));
        SET random_amount = ROUND(RAND() * 1000, 2);
        SET random_date = DATE_SUB(NOW(), INTERVAL FLOOR(RAND() * 730) DAY);

        -- Увеличение вероятности выбора самого плодовитого магазина
        IF random_store_id = most_productive_store_id THEN
            SET most_productive_store_sales_count = most_productive_store_sales_count + 1;
            IF most_productive_store_sales_count > 75000 THEN
                SET random_store_id = FLOOR(1 + (RAND() * 10));
            END IF;
        END IF;

        INSERT INTO sales (store_id, date, sale_amount) VALUES (random_store_id, random_date, random_amount);
        SET trade_counter = trade_counter + 1;
    END WHILE;
END //

DELIMITER ;
```

2. Запрос нарастающего итога продаж по каждому магазину.
```sql
SELECT 
    store_id,
    DATE_FORMAT(date, '%Y-%m') AS month,
    SUM(sale_amount) AS monthly_sales, -- вычисляем сумму продаж за каждый месяц для каждого магазина
    SUM(SUM(sale_amount)) OVER (PARTITION BY store_id ORDER BY DATE_FORMAT(date, '%Y-%m')) AS running_total -- вычисляем нарастающий итог продаж для каждого магазина, группируем данные по месяцам.
FROM 
    sales
GROUP BY 
    store_id, DATE_FORMAT(date, '%Y-%m')
ORDER BY 
    store_id, month;
```

3. 7-дневное скользящее среднее за последний месяц по самому плодовитому магазину.
```sql
WITH last_month_sales AS (
    SELECT 
        store_id,
        SUM(sale_amount) AS total_sales
    FROM 
        sales
    WHERE 
        date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)
    GROUP BY 
        store_id
    ORDER BY 
        total_sales DESC
    LIMIT 1
),
sales_data AS (
    SELECT 
        s.store_id,
        s.date,
        s.sale_amount
    FROM 
        sales s
    JOIN 
        last_month_sales lms ON s.store_id = lms.store_id
    WHERE 
        s.date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)
)
SELECT 
    store_id,
    date,
    sale_amount,
    AVG(sale_amount) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS rolling_avg_7d
FROM 
    sales_data
ORDER BY 
    date;
```

4. Граничные случаи
- по нарастающему итогу: если у магазина нет продаж в выбранном месяце, нарастающий итог будет равен нарастающему итогу предыдущего месяца.
- по скользящему среднему: если у магазина нет продаж в последнем месяце, запрос вернёт пустой результат ("last_month_sales" не найдет ни одной записи, тогда "sales_data" будет пустым, - как следствие, пустой результат основного запроса).
