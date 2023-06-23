1. Посчитайте, сколько компаний закрылось.

```sql
SELECT COUNT(*)
FROM company
WHERE status = 'closed';
```
---

2. Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы company. Отсортируйте таблицу по убыванию значений в поле funding_total .

```sql
SELECT funding_total
FROM company
WHERE category_code = 'news'
AND country_code = 'USA'
ORDER BY funding_total DESC;
```
---

3. Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.

```sql
SELECT SUM(price_amount)
FROM acquisition
WHERE term_code = 'cash'
AND EXTRACT(YEAR FROM acquired_at) IN (2011, 2012, 2013);
```
---

4. Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на 'Silver'.

```sql
SELECT first_name,
       last_name,
       twitter_username
FROM people
WHERE twitter_username LIKE 'Silver%';
```
---

5. Выведите на экран всю информацию о людях, у которых названия аккаунтов в твиттере содержат подстроку 'money', а фамилия начинается на 'K'.

```sql
SELECT *
FROM people
WHERE (twitter_username LIKE '%money%')
AND (last_name LIKE 'K%');
```
---

6. Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.

```sql
SELECT country_code,
       SUM(funding_total) as sum
FROM company
GROUP BY country_code
ORDER BY sum DESC;
```
---

7. Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.

```sql
SELECT funded_at,
       MIN(raised_amount),
       MAX(raised_amount)
FROM funding_round
GROUP BY funded_at
HAVING (MIN(raised_amount) <> 0)
AND (MIN(raised_amount) <>  MAX(raised_amount));
```
---

8. Создайте поле с категориями:
- Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
- Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
- Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.

Отобразите все поля таблицы fund и новое поле с категориями.

```sql
SELECT *,
       CASE
           WHEN invested_companies >=100 THEN 'high_activity'
           WHEN (invested_companies >=20) AND (invested_companies < 100) THEN 'middle_activity'
           WHEN invested_companies < 20 THEN 'low_activity'
       END
FROM fund;
```
---

9. Для каждой из категорий, назначенных в предыдущем задании, посчитайте округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие. Выведите на экран категории и среднее число инвестиционных раундов. Отсортируйте таблицу по возрастанию среднего.

```sql
SELECT CASE
           WHEN invested_companies>=100 THEN 'high_activity'
           WHEN invested_companies>=20 THEN 'middle_activity'
           ELSE 'low_activity'
       END AS activity,
       ROUND(AVG(investment_rounds)) as avg
FROM fund
GROUP BY activity
ORDER BY avg;
```
---

10. Проанализируйте, в каких странах находятся фонды, которые чаще всего инвестируют в стартапы. Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды этой страны, основанные с 2010 по 2012 год включительно. Исключите страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. Выгрузите десять самых активных стран-инвесторов: отсортируйте таблицу по среднему количеству компаний от большего к меньшему. Затем добавьте сортировку по коду страны в лексикографическом порядке.

```sql
SELECT country_code,
       MIN(invested_companies),
       MAX(invested_companies),
       AVG(invested_companies)
FROM fund
WHERE EXTRACT(YEAR FROM founded_at) IN (2010, 2011, 2012)
GROUP BY country_code
HAVING MIN(invested_companies) <> 0
ORDER BY avg DESC, country_code
LIMIT 10;
```
---

11. Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.

```sql
SELECT p.first_name,
       p.last_name,
       ed.instituition
FROM people as p
LEFT JOIN education as ed ON p.id=ed.person_id;
```
---

12. Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.

```sql
WITH cnt AS (SELECT p.company_id,
                    COUNT(DISTINCT ed.instituition)
             FROM people as p
             LEFT JOIN education as ed ON p.id=ed.person_id
             WHERE p.company_id IS NOT NULL
             GROUP BY p.company_id
             ORDER BY count DESC
             LIMIT 5)
SELECT comp.name,
       cnt.count
FROM company as comp
INNER JOIN cnt ON comp.id=cnt.company_id;
```
---

13. Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.

```sql
SELECT name
FROM company
WHERE (status = 'closed')
AND (id IN (SELECT company_id
            FROM funding_round
            WHERE (is_first_round = 1)
            AND (is_last_round = 1)));
```
---

14. Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.

```sql
SELECT DISTINCT p.id
FROM people AS p
WHERE p.company_id IN (SELECT c.id
                       FROM company AS c
                       JOIN funding_round AS fr ON c.id = fr.company_id
                       WHERE STATUS ='closed'
                       AND is_first_round = 1
                       AND is_last_round = 1
                       GROUP BY c.id);
```
---

15. Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.

```sql
WITH idp AS 
(select id
from people
where company_id IN (SELECT id
                     FROM company
                     WHERE (status = 'closed')    
                     AND (id IN (SELECT company_id
                                 FROM funding_round
                                 WHERE (is_first_round = 1)
                                 AND (is_last_round = 1))))) 
SELECT DISTINCT idp.id, 
       inst.instituition
FROM idp 
INNER JOIN education AS inst ON idp.id=inst.person_id;
```
---


16. 
