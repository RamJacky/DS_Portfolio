# Задачи SQL

*Схема базы данных*

![data_base](https://github.com/RamJacky/DS_Portfolio/blob/main/SQL_tasks/db.png)

1. Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы company. Отсортируйте таблицу по убыванию значений в поле funding_total .

```sql
SELECT SUM(funding_total) as total_funding 
FROM company 
WHERE category_code = 'news' AND country_code = 'USA' 
GROUP BY name 
ORDER BY total_funding DESC
```
---

2. Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.

```sql
SELECT SUM(price_amount) as total_deals 
FROM acquisition 
WHERE term_code = 'cash' AND acquired_at BETWEEN '2011-01-01' AND '2013-12-31'
```
---

3. Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на 'Silver'.

```sql
SELECT first_name, last_name, twitter_username
FROM people
WHERE twitter_username LIKE 'Silver%'
```
---

4. Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.

```sql
SELECT country_code, SUM(funding_total) as total_funding
FROM company
GROUP BY country_code
ORDER BY total_funding DESC
```
---

5. Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
  Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.

```sql
SELECT funded_at, MIN(funding_round.raised_amount) as min_funding, MAX(funding_round.raised_amount) as max_funding
FROM funding_round
GROUP BY funded_at
HAVING MIN(funding_round.raised_amount) <> 0 AND MIN(funding_round.raised_amount) <> MAX(funding_round.raised_amount)
```
---

6. Создайте поле с категориями:
  - Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
  - Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
  - Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.
  Отобразите все поля таблицы fund и новое поле с категориями.

```sql
SELECT *, 
CASE
    WHEN invested_companies >= 100 THEN 'high_activity'
    WHEN invested_companies >= 20 AND invested_companies < 100 THEN 'middle_activity'
    ELSE 'low_activity'
END AS activity_category
FROM fund
```
---

7. Проанализируйте, в каких странах находятся фонды, которые чаще всего инвестируют в стартапы. 
  - Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды этой страны, основанные с 2010 по 2012 год включительно. Исключите страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. 
  - Выгрузите десять самых активных стран-инвесторов: отсортируйте таблицу по среднему количеству компаний от большего к меньшему. Затем добавьте сортировку по коду страны в лексикографическом порядке.

```sql
SELECT 
    country_code, 
    MIN(invested_companies) AS min_invested, 
    MAX(invested_companies) AS max_invested, 
    AVG(invested_companies) AS avg_invested
FROM 
    fund
WHERE 
    founded_at BETWEEN '2010-01-01' AND '2012-12-31'
GROUP BY 
    country_code
HAVING 
    MIN(invested_companies) > 0
ORDER BY 
    avg_invested DESC, 
    country_code ASC
LIMIT 10;  
```
---

8. Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.

```sql
SELECT company.name, COUNT(DISTINCT education.instituition) AS universities_count
FROM company
JOIN people ON company.id = people.company_id
JOIN education ON people.id = education.person_id
GROUP BY company.name
ORDER BY universities_count DESC
LIMIT 5;
```
---

9. Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.

```sql
SELECT DISTINCT company.name
FROM funding_round fr1
JOIN funding_round fr2 ON fr1.company_id = fr2.company_id
JOIN company ON fr1.company_id = company.id
WHERE fr1.funded_at = fr2.funded_at
AND fr1.is_first_round = 1 AND fr2.is_last_round = 1 AND company.status = 'closed';
```
---

10. Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.

```sql
SELECT DISTINCT people.id
FROM people
WHERE people.company_id IN (
  SELECT company.id
  FROM company
  JOIN funding_round fr1 ON fr1.company_id = company.id
  JOIN funding_round fr2 ON fr1.company_id = fr2.company_id
  WHERE fr1.funded_at = fr2.funded_at
  AND fr1.is_first_round = 1 AND fr2.is_last_round = 1 AND company.status = 'closed'
);
```
---

11. Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.

```sql
SELECT DISTINCT people.id, education.instituition
FROM people
JOIN education ON people.id = education.person_id
WHERE people.company_id IN (
  SELECT company.id
  FROM company
  JOIN funding_round fr1 ON fr1.company_id = company.id
  JOIN funding_round fr2 ON fr1.company_id = fr2.company_id
  WHERE fr1.funded_at = fr2.funded_at
  AND fr1.is_first_round = 1 AND fr2.is_last_round = 1 AND company.status = 'closed'
);
```
---

12. Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания. При подсчёте учитывайте, что некоторые сотрудники могли окончить одно и то же заведение дважды.

```sql
SELECT DISTINCT people.id, COUNT(education.instituition) as num_education
FROM people
JOIN education ON people.id = education.person_id
WHERE people.company_id IN (
  SELECT company.id
  FROM company
  JOIN funding_round fr1 ON fr1.company_id = company.id
  JOIN funding_round fr2 ON fr1.company_id = fr2.company_id
  WHERE fr1.funded_at = fr2.funded_at
  AND fr1.is_first_round = 1 AND fr2.is_last_round = 1 AND company.status = 'closed'
)
GROUP BY people.id;
```
---

13. Дополните предыдущий запрос и выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.

```sql
SELECT AVG(num_education) as avg_education
FROM (SELECT DISTINCT people.id, COUNT(education.instituition) as num_education
        FROM people
        JOIN education ON people.id = education.person_id
        WHERE people.company_id IN (
            SELECT company.id
            FROM company
            JOIN funding_round fr1 ON fr1.company_id = company.id
            JOIN funding_round fr2 ON fr1.company_id = fr2.company_id
            WHERE fr1.funded_at = fr2.funded_at
            AND fr1.is_first_round = 1 AND fr2.is_last_round = 1 AND company.status = 'closed'
            )
GROUP BY people.id
) as subquery;
```
---

14. Составьте таблицу из полей:
  - name_of_fund — название фонда;
  - name_of_company — название компании;
  - amount — сумма инвестиций, которую привлекла компания в раунде.
  - В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.

```sql
SELECT fund.name AS name_of_fund, company.name AS name_of_company, funding_round.raised_amount AS amount
FROM investment
JOIN funding_round ON investment.funding_round_id = funding_round.id
JOIN company ON funding_round.company_id = company.id
JOIN fund ON investment.fund_id = fund.id
WHERE company.milestones > 6 
AND funding_round.funded_at BETWEEN '2012-01-01' AND '2013-12-31';
```
---

15. Выгрузите таблицу, в которой будут такие поля:
  - название компании-покупателя;
  - сумма сделки;
  - название компании, которую купили;
  - сумма инвестиций, вложенных в купленную компанию;
  - доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.
  Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы. 
  Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в лексикографическом порядке. Ограничьте таблицу первыми десятью записями.

```sql
SELECT acquiring.name AS acquiring_company_name, 
       acquisition.price_amount AS acquisition_price,
       acquired.name AS acquired_company_name,
       acquired.funding_total AS acquired_funding_total,
       ROUND(acquisition.price_amount / NULLIF(acquired.funding_total, 0)) AS acquisition_to_funding_ratio
FROM acquisition
JOIN company AS acquiring ON acquisition.acquiring_company_id = acquiring.id
JOIN company AS acquired ON acquisition.acquired_company_id = acquired.id
WHERE acquisition.price_amount != 0 AND acquired.funding_total != 0
ORDER BY acquisition.price_amount DESC, acquired_company_name
LIMIT 10;
```
---

16. Выгрузите таблицу, в которую войдут названия компаний из категории social, получившие финансирование с 2010 по 2013 год включительно. Проверьте, что сумма инвестиций не равна нулю. Выведите также номер месяца, в котором проходил раунд финансирования.

```sql
SELECT c.name, EXTRACT(MONTH FROM f.funded_at) AS month_number
FROM company c
JOIN funding_round f ON c.id = f.company_id
WHERE c.category_code = 'social'
AND f.funded_at BETWEEN '2010-01-01' AND '2013-12-31'
AND f.raised_amount > 0
ORDER BY month_number;
```
---

17. Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
  - номер месяца, в котором проходили раунды;
  - количество уникальных названий фондов из США, которые инвестировали в этом месяце;
  - количество компаний, купленных за этот месяц;
  - общая сумма сделок по покупкам в этом месяце.

```sql
WITH 
funds AS
(SELECT EXTRACT (MONTH FROM fr.funded_at) AS month,
       COUNT(DISTINCT f.name) AS count_of_funds_usa 
       FROM funding_round AS fr 
       JOIN investment AS i ON fr.id=i.funding_round_id
       JOIN fund AS f ON i.fund_id=f.id WHERE EXTRACT(YEAR FROM fr.funded_at) BETWEEN 2010 AND 2013
       AND f.country_code = 'USA' 
       GROUP by month),
acq AS
(SELECT EXTRACT (MONTH FROM a.acquired_at) AS month,                   
        COUNT(a.acquired_company_id) AS count_of_acquired,
        SUM(a.price_amount) AS sum_of_acquired FROM acquisition AS a 
        WHERE EXTRACT (YEAR FROM a.acquired_at) BETWEEN 2010 AND 2013
GROUP by month)
SELECT fs.month,count_of_funds_usa,
       count_of_acquired,      
       sum_of_acquired       
FROM funds AS fs 
JOIN acq AS aq ON fs.month=aq.month
```
---

18. Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.

```sql
WITH
invest_2011 AS (SELECT AVG(funding_total) as avg_funding_total,
                       country_code
                FROM company
                WHERE EXTRACT(YEAR FROM CAST(founded_at AS TIMESTAMP)) = 2011
                      --AND id IN (SELECT DISTINCT company_id FROM investment)
               GROUP BY country_code),
invest_2012 AS (SELECT AVG(funding_total) as avg_funding_total,
                       country_code
                FROM company
                WHERE EXTRACT(YEAR FROM CAST(founded_at AS TIMESTAMP)) = 2012
                      --AND id IN (SELECT DISTINCT company_id FROM investment)
               GROUP BY country_code),
invest_2013 AS (SELECT AVG(funding_total) as avg_funding_total,
                       country_code
                FROM company
                WHERE EXTRACT(YEAR FROM CAST(founded_at AS TIMESTAMP)) = 2013
                      --AND id IN (SELECT DISTINCT company_id FROM investment)
               GROUP BY country_code)
                      
SELECT i1.country_code,
       i1.avg_funding_total as q1,
       i2.avg_funding_total as q2,
       i3.avg_funding_total as q3
FROM invest_2011 i1
         JOIN invest_2012 i2 ON i1.country_code = i2.country_code
         JOIN invest_2013 i3 ON i1.country_code = i3.country_code
ORDER BY q1 DESC;
```
---

19. Напишите запрос, который проранжирует расходы на привлечение пользователей за каждый день по убыванию. 
  Выгрузите три поля: 
  - дата, которую нужно привести к типу date;
  - расходы на привлечение;
  - ранг строки.

```sql
select cast(created_at as date),
       costs,
       row_number() over( order by costs desc)
from tools_shop.costs;
```
---

20. Измените предыдущий запрос: записям с одинаковыми значениями расходов назначьте одинаковый ранг. Ранги не должны прерываться.

```sql
select cast(created_at as date),
       costs,
       dense_rank() over( order by costs desc)
from tools_shop.costs;
```
