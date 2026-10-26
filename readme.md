## Фінальний проєкт (Final Project): База даних `pandemic`

### Завдання 1. Завантажте дані. (p1_Anhelov.png)

Створіть схему pandemic у базі даних за допомогою SQL-команди.
Оберіть її як схему за замовчуванням за допомогою SQL-команди.
Імпортуйте дані за допомогою Import wizard так, як ви вже робили це у темі 3.
Продивіться дані, щоб бути у контексті.
Атрибути Entity та Code постійно повторюються. Позбудьтеся цього за допомогою нормалізації даних.

```sql
CREATE SCHEMA IF NOT EXISTS pandemic;
USE pandemic;
SELECT * FROM infectious_cases LIMIT 10;
```

### Завдання 2. Нормалізуйте таблицю infectious_cases до 3ї нормальної форми. Збережіть у цій же схемі дві таблиці з нормалізованими даними.(p2_1-2-4_Anhelov.png)

Виконайте запит SELECT COUNT(\*) FROM infectious_cases , щоб ментор міг зрозуміти, скільки записів ви завантажили у базу даних із файла.

```sql
USE pandemic;

SELECT COUNT(*) AS total_rows FROM infectious_cases;

CREATE TABLE entities (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity VARCHAR(255),
    code VARCHAR(50)
);

INSERT INTO entities (entity, code)
SELECT DISTINCT Entity, Code
FROM infectious_cases;

CREATE TABLE normalized_infectious_cases (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity_id INT,
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
    FOREIGN KEY (entity_id) REFERENCES entities(id)
);

INSERT INTO normalized_infectious_cases (
    entity_id, Year, Number_yaws, polio_cases, cases_guinea_worm,
    Number_rabies, Number_malaria, Number_hiv, Number_tuberculosis,
    Number_smallpox, Number_cholera_cases
)
SELECT
    e.id, ic.Year, ic.Number_yaws, ic.polio_cases, ic.cases_guinea_worm,
    ic.Number_rabies, ic.Number_malaria, ic.Number_hiv, ic.Number_tuberculosis,
    ic.Number_smallpox, ic.Number_cholera_cases
FROM infectious_cases ic
JOIN entities e ON ic.Entity = e.entity AND (ic.Code = e.code OR (ic.Code IS NULL AND e.code IS NULL));
```

### Завдання 3. Проаналізуйте дані.(p3_Anhelov.png)

Для кожної унікальної комбінації Entity та Code або їх id порахуйте середнє, мінімальне, максимальне значення та суму для атрибута Number_rabies.
Врахуйте, що атрибут Number_rabies може містити порожні значення ‘’ — вам попередньо необхідно їх відфільтрувати.
Результат відсортуйте за порахованим середнім значенням у порядку спадання.
Оберіть тільки 10 рядків для виведення на екран.

```sql
USE pandemic;

SELECT
    e.entity,
    e.code,
    AVG(CAST(nic.Number_rabies AS DECIMAL(10,4))) AS avg_rabies,
    MIN(CAST(nic.Number_rabies AS DECIMAL(10,4))) AS min_rabies,
    MAX(CAST(nic.Number_rabies AS DECIMAL(10,4))) AS max_rabies,
    SUM(CAST(nic.Number_rabies AS DECIMAL(10,4))) AS sum_rabies
FROM normalized_infectious_cases nic
JOIN entities e ON nic.entity_id = e.id
WHERE nic.Number_rabies IS NOT NULL
  AND nic.Number_rabies != ''
GROUP BY e.entity, e.code
ORDER BY avg_rabies DESC
LIMIT 10;
```

### Завдання 4. Побудуйте колонку різниці в роках.(p4_Anhelov.png)

Для оригінальної або нормованої таблиці для колонки Year побудуйте з використанням вбудованих SQL-функцій:
атрибут, що створює дату першого січня відповідного року,
Наприклад, якщо атрибут містить значення ’1996’, то значення нового атрибута має бути ‘1996-01-01’.
атрибут, що дорівнює поточній даті,
атрибут, що дорівнює різниці в роках двох вищезгаданих колонок.
Перераховувати всі інші атрибути, такі як Number_malaria, не потрібно.

```sql
USE pandemic;

SELECT
    id,
    Year,
    CAST(CONCAT(Year, '-01-01') AS DATE) AS start_of_year,
    CURRENT_DATE() AS current_date_col,
    TIMESTAMPDIFF(YEAR, CAST(CONCAT(Year, '-01-01') AS DATE), CURRENT_DATE()) AS year_difference
FROM normalized_infectious_cases
LIMIT 20;
```

### Завдання 5. Побудуйте власну функцію.(p5_Anhelov.png)

Створіть і використайте функцію, що будує такий же атрибут, як і в попередньому завданні: функція має приймати на вхід значення року, а повертати різницю в роках між поточною датою та датою, створеною з атрибута року (1996 рік → ‘1996-01-01’).

```sql
USE pandemic;

DROP FUNCTION IF EXISTS CalculateYearDifference;

DELIMITER //

CREATE FUNCTION CalculateYearDifference(input_year INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE calculated_diff INT;
    SET calculated_diff = TIMESTAMPDIFF(YEAR, CAST(CONCAT(input_year, '-01-01') AS DATE), CURRENT_DATE());
    RETURN calculated_diff;
END //

DELIMITER ;

SELECT
    id,
    Year,
    CalculateYearDifference(Year) AS calculated_year_diff
FROM normalized_infectious_cases
LIMIT 20;
```
