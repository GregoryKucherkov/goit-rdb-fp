# goit-rdb-fp

#1. Завантажте дані:

Створіть схему pandemic у базі даних за допомогою SQL-команди.
Оберіть її як схему за замовчуванням за допомогою SQL-команди.
Імпортуйте дані за допомогою Import wizard так, як ви вже робили це у темі 3.

код:
CREATE DATABASE IF NOT EXISTS pandemic;
USE pandemic;

#2. Нормалізуйте таблицю infectious_cases до 3ї нормальної форми. Збережіть у цій же схемі дві таблиці з нормалізованими даними.

код:

спочатку подивитися які datatype у таблиці
DESCRIBE infectious_cases;

потім створюємо і наповнюємо:
USE pandemic;

CREATE TABLE IF NOT EXISTS entity_code (
	id INT AUTO_INCREMENT PRIMARY KEY,
    entity VARCHAR(255),
    code VARCHAR(255)
);

INSERT INTO entity_code (entity, code) 
SELECT DISTINCT entity, code
FROM
	infectious_cases;
    
CREATE TABLE IF NOT EXISTS infectious_cases_norm (
	entity_code_id INT,
    Year INT, 
    Number_yaws TEXT,
	polio_cases TEXT,
	cases_guinea_worm TEXT,
	Number_rabies TEXT,
	Number_malaria TEXT,
	Number_hiv TEXT,
	Number_tuberculosis TEXT,
	Number_smallpox TEXT,
	Number_cholera_cases TEXT,
    
PRIMARY KEY (entity_code_id, Year),
FOREIGN KEY (entity_code_id) REFERENCES entity_code(id)
);

INSERT INTO infectious_cases_norm(
	entity_code_id, Year, Number_yaws, polio_cases, cases_guinea_worm, Number_rabies, Number_malaria, Number_hiv, Number_tuberculosis, Number_smallpox,
    Number_cholera_cases
)
SELECT
ec.id AS entity_code_id,
ic.Year,
ic.Number_yaws,
ic.polio_cases,
ic.cases_guinea_worm, 
ic.Number_rabies, 
ic.Number_malaria, 
ic.Number_hiv, 
ic.Number_tuberculosis, 
ic.Number_smallpox,
ic.Number_cholera_cases
FROM
	infectious_cases ic
JOIN
	entity_code ec
ON 
	ic.entity = ec.entity AND ic.code = ec.code;

#3. Проаналізуйте дані:
- Для кожної унікальної комбінації Entity та Code або їх id порахуйте середнє, мінімальне, максимальне значення та суму для атрибута Number_rabies.
💡 Врахуйте, що атрибут Number_rabies може містити порожні значення ‘’ — вам попередньо необхідно їх відфільтрувати.

- Результат відсортуйте за порахованим середнім значенням у порядку спадання.
- Оберіть тільки 10 рядків для виведення на екран.

код:
SELECT entity_code_id, avg(Number_rabies) Avg, min(Number_rabies) Min, max(Number_rabies) Max, sum(Number_rabies) Sum
FROM 
	infectious_cases_norm
WHERE 
	Number_rabies > 0
GROUP BY
	entity_code_id;

 ORDER BY Avg DSC
 
 LIMIT 10;

#4. Побудуйте колонку різниці в роках.

Для оригінальної або нормованої таблиці для колонки Year побудуйте з використанням вбудованих SQL-функцій:

атрибут, що створює дату першого січня відповідного року,
💡 Наприклад, якщо атрибут містить значення ’1996’, то значення нового атрибута має бути ‘1996-01-01’.
атрибут, що дорівнює поточній даті,
атрибут, що дорівнює різниці в роках двох вищезгаданих колонок.

код:
USE pandemic;

4.1
SELECT *, Makedate(Year, 1) AS Date_atribute
FROM
  infectious_cases_norm;

4.2
SELECT *, CURDATE() AS `Current_date`
FROM
  infectious_cases_norm;

4.3
SELECT Year,
	-- option 1
    Year(CURDATE()) - Year AS `Date_difference (years)`,
    -- option 2
    DATEDIFF(CURDATE(), MAKEDATE(Year, 1)) / 365 AS `Date_difference_2 (years)`
FROM
	infectious_cases_norm;

#5 Побудуйте власну функцію.

Створіть і використайте функцію, що будує такий же атрибут, як і в попередньому завданні: функція має приймати на вхід значення року, а повертати різницю в роках між поточною датою та датою, створеною з атрибута року (1996 рік → ‘1996-01-01’).

код:
DROP FUNCTION IF EXISTS Years_difference;

DELIMITER //

CREATE FUNCTION Years_difference(input_year INT)
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN 
	DECLARE result DECIMAL(10, 2);
	SET result = DATEDIFF(CURDATE(), MAKEDATE(Year, 1)) / 365;
    -- Alternative logic for years only difference without fractions
    -- SET result = Year(CURDATE()) - input_year; 
    RETURN result;
END //

DELIMITER ;

Usage:

SELECT Year, 
  Years_difference(Year) AS `Date_difference (years)`
FROM
  infectious_cases_norm;

5.2
Створіть функцію, що рахує кількість захворювань за певний період. Для цього треба поділити кількість захворювань на рік на певне число: 12 — для отримання середньої кількості захворювань на місяць, 4 — на квартал або 2 — на півріччя. Таким чином, функція буде приймати два параметри: кількість захворювань на рік та довільний дільник. Ви також маєте використати її — запустити на даних. Оскільки не всі рядки містять число захворювань, вам необхідно буде відсіяти ті, що не мають чисельного значення (≠ ‘’).

код:
DROP FUNCTION IF EXISTS Diseases_for_period_1;

DELIMITER //

CREATE FUNCTION Diseases_for_period_1(input_year INT, period INT)
RETURNS DECIMAL(10, 4)
DETERMINISTIC
BEGIN 
	DECLARE total_cases DECIMAL(10, 4);
	SELECT SUM(polio_cases) 
    INTO total_cases
    FROM infectious_cases_norm
    WHERE Year = input_year AND
		polio_cases > 0 ;
    RETURN total_cases / period;
END //

DELIMITER ;

Usage:
USE pandemic;

SELECT Year,  
	Diseases_for_period(SUM(polio_cases), 6) AS `Diseases for a period`
FROM
	infectious_cases_norm
WHERE Year = 1988
	AND polio_cases IS NOT NULL 
    AND polio_cases != '';

  



