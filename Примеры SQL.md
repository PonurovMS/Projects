# SQL
## Существуют следующие таблицы с данными:
📁 Таблица payments

id_client (уникальный идентификатор клиента)

time_payment (дата и время платежа в формате "гггг-мм-дд чч:мм:сс")

amt_payment (размер платежа)

📁 Таблица client_info

id_client (уникальный идентификатор клиента)

gender (пол клиента)

age (возраст клиента)

id_city (идентификатор города клиента)

📁 Таблица city_info

id_city (идентификатор города клиента)

name_city (название города клиента)

name_region (наименование федерального округа, в котором расположен данный город)


## Задание 1.
Для каждого города найдите долю мужчин (% мужчин среди всех клиентов в данном городе). Ограничьтесь только клиентами, которым от 20 до 40 лет. В выводе используйте названия городов, а не идентификаторы.

```
WITH		t AS
(
SELECT		a.name_city, COUNT(gender) AS male
FROM		city_info AS a join client_info AS b ON a.id_city = b.id_city
WHERE		b.age BETWEEN '20' AND '40' AND gender = 'M'
GROUP BY 	1
), t2 AS
(
SELECT		a.name_city, COUNT(gender) AS all_gender
FROM		city_info AS a JOIN client_info AS b ON a.id_city = b.id_city
WHERE		b.age BETWEEN '20' AND '41'
GROUP BY 	1
)
SELECT		 t.name_city, ROUND(t.male * 1.0/ t2.all_gender*100, 2) AS percent_male
FROM		 t JOIN t2 ON t.name_city = t2.name_city 
```

## Задание 2.
Определите средний возраст по тем клиентам, которые ни разу ничего не заплатили.

```
SELECT		AVG(a.age)
FROM		client_info AS a LEFT JOIN payments AS b ON a.id_client = b.id_client
WHERE		b.amt_payment IS NULL
```
## Задание 3.
Для каждого федерального округа выделите первые три платежа.

```
SELECT *
FROM
(
SELECT		a.amt_payment, a.time_payment, c.name_region
, ROW_NUMBER() OVER (PARTITION BY c.name_region order by time_payment asc) AS num
FROM		payments AS a join client_info AS b ON a.id_client = b.id_client
JOIN city_info AS c ON b.id_city = c.id_city
 )
 WHERE num < 4
 ```
 
## Задание 4.
Ограничьтесь клиентами из федеральных округов "Южный" и "Северный". Для каждого города рассчитайте, сколько в среднем времени проходит между платежами одного клиента.
```
SELECT 		name_city, avg(i)
FROM
(
SELECT		a.id_client, (LEAD(a.time_payment) OVER (PARTITION BY a.id_client ORDER BY a.time_payment))-
  		a.time_payment AS i, c.name_city
FROM		payments AS a
		JOIN client_info AS b ON a.id_client = b.id_client
		JOIN city_info AS c ON b.id_city = c.id_city
WHERE		name_region = 'Южный' OR name_region = 'Северный'
)
WHERE 	i IS NOT NULL
GROUP BY 	1
```
