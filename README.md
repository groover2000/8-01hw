### Домашнее задание к занятию «SQL. Часть 2»

## Задание 1
Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию:

фамилия и имя сотрудника из этого магазина;
город нахождения магазина;
количество пользователей, закреплённых в этом магазине.

```SQL
SELECT COUNT(s.store_id) AS users_count, CONCAT_WS(' ', s2.first_name, s2.last_name), c2.city
FROM sakila.store s 
LEFT JOIN sakila.customer c ON c.store_id = s.store_id
INNER JOIN sakila.staff s2 ON c.store_id = s2.store_id
INNER JOIN sakila.address a ON s2.address_id = a.address_id
INNER JOIN sakila.city c2 ON c2.city_id = a.city_id
GROUP BY s.manager_staff_id, s2.first_name, s2.last_name, c2.city
HAVING users_count > 300
```
<center>
	<img src='./img/sql1.png'>
</center>

## Задание 2
Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.

```SQL
SELECT COUNT(f.`length`)
FROM sakila.film f
WHERE f.`length` > (
	SELECT AVG(f2.`length`)
	FROM sakila.film f2 
)
```
<center>
	<img src='./img/sql2.png'>
</center>

## Задание 3
Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.

```SQL
SELECT MONTHNAME(r.rental_date) as mname, SUM(p.amount) as sum, COUNT(r.rental_id)
FROM sakila.rental r
INNER JOIN sakila.payment p ON r.rental_id  = p.rental_id
GROUP BY mname
ORDER BY sum DESC
LIMIT 1
```
<center>
	<img src='./img/sql3.png'>
</center>


Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

## Задание 4*
Посчитайте количество продаж, выполненных каждым продавцом. Добавьте вычисляемую колонку «Премия». Если количество продаж превышает 8000, то значение в колонке будет «Да», иначе должно быть значение «Нет».

```SQL
SELECT s.first_name, COUNT(p.payment_id) as payment_count,
CASE
	WHEN COUNT(p.payment_id) > 8000 THEN 'Да'
	ELSE 'Нет'
END AS 'Премия'
FROM sakila.staff s
INNER JOIN sakila.payment p ON p.staff_id = s.staff_id
GROUP BY s.first_name
```
<center>
	<img src='./img/sql4.png'>
</center>

## Задание 5*
Найдите фильмы, которые ни разу не брали в аренду.