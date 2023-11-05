# Задачи SQL

*Схема базы данных*

![242782466-6149324a-72d3-4410-94c7-29fb626f03cc](https://github.com/elena-iliushina/Portfolio/assets/133641038/22e12838-fe98-400c-9d4a-bfc0cba1be92)

1. Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы company. Отсортируйте таблицу по убыванию значений в поле funding_total .

```sql
SELECT funding_total
FROM company
WHERE category_code = 'news' 
      AND country_code = 'USA'
ORDER BY funding_total DESC;
```
---

2. Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.

```sql
SELECT SUM(price_amount)
FROM acquisition
WHERE term_code = 'cash'
      AND EXTRACT(YEAR FROM CAST(acquired_at AS TIMESTAMP)) BETWEEN 2011 AND 2013;
```
---

3. Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на 'Silver'.

```sql
SELECT first_name,
       last_name,
       twitter_username
FROM people
WHERE twitter_username LIKE 'Silver%';
```
---

4. Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.

```sql
SELECT country_code,
       SUM(funding_total) AS country_funding_total
FROM company
GROUP BY country_code
ORDER BY country_funding_total DESC;
```
---

5. Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
  Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.

```sql
SELECT funded_at,
       MAX(raised_amount) AS max_raised_amount,
       MIN(raised_amount) AS min_raised_amount
FROM funding_round
GROUP BY funded_at
HAVING MIN(raised_amount) != 0 AND MIN(raised_amount) != MAX(raised_amount);
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
           WHEN invested_companies < 20 THEN 'low_activity'
           WHEN invested_companies >= 20 AND invested_companies < 100 THEN 'middle_activity'
           WHEN invested_companies >= 100 THEN 'high_activity'
       END AS category_invested_companies
FROM fund;
```
---

7. Проанализируйте, в каких странах находятся фонды, которые чаще всего инвестируют в стартапы. 
  - Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды этой страны, основанные с 2010 по 2012 год включительно. Исключите страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. 
  - Выгрузите десять самых активных стран-инвесторов: отсортируйте таблицу по среднему количеству компаний от большего к меньшему. Затем добавьте сортировку по коду страны в лексикографическом порядке.

```sql
SELECT country_code,
       MIN(invested_companies) AS min_invested_companies,
       MAX(invested_companies) AS max_invested_companies,
       AVG(invested_companies) AS avg_invested_companies
FROM fund
WHERE EXTRACT(YEAR FROM CAST(founded_at AS TIMESTAMP)) BETWEEN 2010 AND 2012
GROUP BY country_code
HAVING MIN(invested_companies) != 0
ORDER BY avg_invested_companies DESC, country_code
LIMIT 10;
```
---

8. Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.

```sql
SELECT c.name,
       COUNT(DISTINCT e.instituition) AS count_instituition
FROM company c
     INNER JOIN people p ON p.company_id = c.id
     INNER JOIN education e ON e.person_id = p.id
GROUP BY c.name
ORDER BY count_instituition DESC
LIMIT 5;
```
---

9. Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.

```sql
SELECT DISTINCT name
FROM company
WHERE status = 'closed'
      AND id IN (SELECT DISTINCT company_id
                 FROM funding_round
                 WHERE is_first_round = 1 
                       AND is_last_round = 1);
```
---

10. Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.

```sql
WITH
closed_company AS (SELECT DISTINCT id as id
                    FROM company
                    WHERE status = 'closed'
                          AND id IN (SELECT DISTINCT company_id
                                     FROM funding_round
                                     WHERE is_first_round = 1 
                                           AND is_last_round = 1))

SELECT DISTINCT id
FROM people
WHERE company_id IN (SELECT id
                     FROM closed_company);
```
---

11. Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.

```sql
WITH
closed_company AS (SELECT DISTINCT id as id
                    FROM company
                    WHERE status = 'closed'
                          AND id IN (SELECT DISTINCT company_id
                                     FROM funding_round
                                     WHERE is_first_round = 1 
                                           AND is_last_round = 1))

SELECT DISTINCT p.id,
                e.instituition
FROM people p
     JOIN education e ON p.id = e.person_id
WHERE p.company_id IN (SELECT id
                     FROM closed_company);
```
---

12. Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания. При подсчёте учитывайте, что некоторые сотрудники могли окончить одно и то же заведение дважды.

```sql
WITH
closed_company AS (SELECT DISTINCT id as id
                    FROM company
                    WHERE status = 'closed'
                          AND id IN (SELECT DISTINCT company_id
                                     FROM funding_round
                                     WHERE is_first_round = 1 
                                           AND is_last_round = 1))

SELECT p.id,
       COUNT(e.id)
FROM people p
     JOIN education e ON p.id = e.person_id
WHERE p.company_id IN (SELECT id
                     FROM closed_company)
GROUP BY p.id;
```
---

13. Дополните предыдущий запрос и выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.

```sql
WITH
closed_company AS (SELECT DISTINCT id as id
                    FROM company
                    WHERE status = 'closed'
                          AND id IN (SELECT DISTINCT company_id
                                     FROM funding_round
                                     WHERE is_first_round = 1 
                                           AND is_last_round = 1))

SELECT AVG(count_education)
FROM (SELECT p.id,
             COUNT(e.id) AS count_education
       FROM people p
       JOIN education e ON p.id = e.person_id
       WHERE p.company_id IN (SELECT id
                              FROM closed_company)
       GROUP BY p.id) as a;
```
---

14. Составьте таблицу из полей:
  - name_of_fund — название фонда;
  - name_of_company — название компании;
  - amount — сумма инвестиций, которую привлекла компания в раунде.
  - В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.

```sql
WITH
main_company AS (SELECT id,
                        name
                 FROM company
                 WHERE milestones > 6),
main_fund AS (SELECT id,
                     company_id,
                     raised_amount
              FROM funding_round
              WHERE EXTRACT(YEAR FROM CAST(funded_at AS TIMESTAMP)) BETWEEN 2012 AND 2013)

SELECT DISTINCT 
       f.name AS name_of_fund,
       mc.name AS name_of_company,
       mf.raised_amount AS amount
FROM fund f 
     JOIN investment i ON i.fund_id = f.id
     JOIN main_company mc ON mc.id = i.company_id
     JOIN main_fund mf ON mf.id = i.funding_round_id;
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
       asq.price_amount AS price_amount,
       acquired.name AS acquired_company_name,
       acquired.funding_total AS funding_total,
       ROUND(price_amount/acquired.funding_total) AS price_for_funding
FROM acquisition asq
     JOIN company acquiring ON acquiring.id = asq.acquiring_company_id
     JOIN company acquired ON acquired.id = asq.acquired_company_id
WHERE price_amount != 0 AND acquired.funding_total !=0
ORDER BY price_amount DESC, acquired_company_name
LIMIT 10;
```
---

16. Выгрузите таблицу, в которую войдут названия компаний из категории social, получившие финансирование с 2010 по 2013 год включительно. Проверьте, что сумма инвестиций не равна нулю. Выведите также номер месяца, в котором проходил раунд финансирования.

```sql
WITH 
social_company AS (SELECT name,
                          id
                    FROM company
                    WHERE category_code = 'social' AND funding_total != 0)
                    
SELECT sc.name,
       EXTRACT(MONTH FROM CAST(fr.funded_at AS TIMESTAMP))
FROM funding_round fr 
     JOIN social_company sc ON sc.id = fr.company_id
WHERE EXTRACT(YEAR FROM CAST(fr.funded_at AS TIMESTAMP)) BETWEEN 2010 AND 2013
      AND fr.raised_amount != 0;
```
---

17. Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
  - номер месяца, в котором проходили раунды;
  - количество уникальных названий фондов из США, которые инвестировали в этом месяце;
  - количество компаний, купленных за этот месяц;
  - общая сумма сделок по покупкам в этом месяце.

```sql
WITH
one AS (SELECT EXTRACT(MONTH FROM CAST(fr.funded_at AS TIMESTAMP)) AS month_round,
               COUNT(DISTINCT f.name) AS count_name
         FROM funding_round fr
              JOIN investment i ON fr.id = i.funding_round_id
              JOIN fund f ON f.id = i.fund_id
         WHERE EXTRACT(YEAR FROM CAST(fr.funded_at AS TIMESTAMP)) BETWEEN 2010 AND 2013
               AND f.country_code = 'USA'
         GROUP BY month_round),
two AS (SELECT EXTRACT(MONTH FROM CAST(a.acquired_at AS TIMESTAMP)) AS month_sell,
               COUNT( a.acquired_company_id) AS count_company_id,
               SUM(a.price_amount) AS price_amount
        FROM acquisition a 
        WHERE EXTRACT(YEAR FROM CAST(acquired_at AS TIMESTAMP)) BETWEEN 2010 AND 2013
        GROUP BY month_sell)

SELECT month_round,
       count_name,
       count_company_id,
       price_amount
FROM one JOIN two ON one.month_round = two.month_sell;
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
---

21. Используя оконную функцию, выведите список уникальных user_id пользователей, которые совершили три заказа и более.

```sql
with
users as (select *, row_number() over(partition by user_id order by paid_at) as rn
             from tools_shop.orders)
             
select distinct user_id
from users
where rn = 3;
```
---

22. Рассчитайте количество зарегистрированных пользователей по месяцам с накоплением.
  Выгрузите два поля:
  - месяц регистрации, приведённый к типу date;
  - общее количество зарегистрированных пользователей на текущий месяц.

```sql
select distinct cast(date_trunc('month', created_at) as date),
       count(user_id) over(order by cast(date_trunc('month', created_at) as date) )
from tools_shop.users;
```
---

23. Рассчитайте сумму трат на привлечение пользователей с накоплением по месяцам c 2017 по 2018 год включительно.
  Выгрузите два поля:
  - месяц, приведённый к типу date;
  - сумма трат на текущий месяц с накоплением.

```sql
select distinct cast(date_trunc('month', created_at) as date),
       sum(costs) over(order by cast(date_trunc('month', created_at) as date) )
from tools_shop.costs
where extract(year from cast(created_at as timestamp)) between 2017 and 2018;
```
---

24. Посчитайте события с названием view_item по месяцам с накоплением. Рассчитайте количество событий только для тех пользователей, которые совершили хотя бы одну покупку.
  Выгрузите поля: 
  - месяц события, приведённый к типу date;
  - количество событий за текущий месяц;
  - количество событий за текущий месяц с накоплением.

```sql
select distinct cast(date_trunc('month', event_time) as date),
       count(event_id) over my_window as abc,
       count(event_id) over w2 as def
from tools_shop.events
where event_name = 'view_item' and user_id in (select user_id from tools_shop.orders)
window my_window AS (partition by cast(date_trunc('month', event_time) as date)),
       w2 AS (order by cast(date_trunc('month', event_time) as date));
```
---

25. Используя конструкцию WINDOW, рассчитайте суммарную стоимость и количество заказов с накоплением от месяца к месяцу.
  Выгрузите поля:
  - идентификатор заказа;
  - месяц оформления заказа, приведённый к типу date;
  - сумма заказа;
  - количество заказов с накоплением;
  - суммарная стоимость заказов с накоплением.

```sql
select distinct order_id,
       cast(date_trunc('month', created_at) as date),
       total_amt,
       count(order_id) over my_window as abc,
       sum(total_amt) over w2 as def
from tools_shop.orders
window my_window AS (order by cast(date_trunc('month', created_at) as date)),
       w2 AS (order by cast(date_trunc('month', created_at) as date));
```
---

26. Напишите запрос, который выведет сумму трат на привлечение пользователей по месяцам, а также разницу в тратах между текущим и предыдущим месяцами. Разница должна показывать, на сколько траты текущего месяца отличаются от предыдущего. В случае, если данных по предыдущему месяцу нет, укажите ноль.
  Выгрузите поля:
  - месяц, приведённый к типу date;
  - траты на привлечение пользователей в текущем месяце;
  - разница в тратах между текущим и предыдущим месяцами.

```sql
with
    table_1 as (select  distinct
                       cast(date_trunc('month', created_at) as date) as my_date,
                       sum(costs) over my_window as month_cost
                    from tools_shop.costs
                    WINDOW my_window AS (partition by cast(date_trunc('month', created_at) as date)))

select my_date,
       month_cost,
      month_cost - LAG(month_cost, 1, month_cost) OVER (order BY my_date )
from table_1;
```
---

27. Напишите запрос, который выведет сумму выручки по годам и разницу выручки между текущим и следующим годом. Разница должна показывать, на сколько выручка следующего года отличается от текущего. В случае, если данных по следующему году нет, укажите ноль.
  Выгрузите поля:
  - год, приведённый к типу date;
  - выручка за текущий год;
  - разница в выручке между текущим и следующим годом.

```sql
with
    table_1 as (select  distinct
                       cast(date_trunc('year', paid_at) as date) as my_date,
                       sum(total_amt) over my_window as month_cost
                    from tools_shop.orders
                    WINDOW my_window AS (partition by cast(date_trunc('year', paid_at) as date)))

select my_date,
       month_cost,
      lead(month_cost, 1, month_cost) OVER (order BY my_date) - month_cost
from table_1;
```
---
