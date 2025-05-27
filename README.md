### Домашнее задание к занятию «Индексы»

## Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

```SQL
	SELECT ROUND(SUM(t.INDEX_LENGTH ) / SUM(t.DATA_LENGTH + t.INDEX_LENGTH) * 100, 0) AS answer
	FROM information_schema.TABLES t 
	WHERE t.TABLE_SCHEMA = "sakila"
```

## Задание 2
Выполните explain analyze следующего запроса:

```SQL
	select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
	from payment p, rental r, customer c, inventory i, film f
	where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id

```
Запрос
---
```SQL
SELECT 
  CONCAT(c.last_name, ' ', c.first_name) AS full_name,
  SUM(p.amount) AS total_amount
FROM payment p
JOIN rental r ON p.rental_id = r.rental_id
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
WHERE p.payment_date >= '2005-07-30' AND p.payment_date < '2005-07-31'
GROUP BY full_name, c.last_name
```
 1. Замена каста на between, дает использовать индекс
 2. GROUP BY вместо DISTINCT, берется меньше строк
 3. Немного не понял смысла вообще брать таблиц film
 4. Явное соединение вроде как лучше чем не явное
 5. Возможно добавить индекс для даты

+ перечислите узкие места;
+ оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

Было
---
```SQL
	-> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=4011..4011 rows=391 loops=1)
		-> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=4011..4011 rows=391 loops=1)
			-> Window aggregate with buffering: sum(p.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=1812..3863 rows=642000 loops=1)
				-> Sort: c.customer_id, f.title  (actual time=1811..1856 rows=642000 loops=1)
					-> Stream results  (cost=16.6e+6 rows=17.1e+6) (actual time=14.6..1288 rows=642000 loops=1)
						-> Nested loop inner join  (cost=16.6e+6 rows=17.1e+6) (actual time=3.59..1104 rows=642000 loops=1)
							-> Nested loop inner join  (cost=14.9e+6 rows=17.1e+6) (actual time=3.59..988 rows=642000 loops=1)
								-> Nested loop inner join  (cost=13.2e+6 rows=17.1e+6) (actual time=3.58..867 rows=642000 loops=1)
									-> Inner hash join (no condition)  (cost=1.65e+6 rows=16.5e+6) (actual time=3.48..33.6 rows=634000 loops=1)
										-> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.72 rows=16500) (actual time=0.203..4.28 rows=634 loops=1)
											-> Table scan on p  (cost=1.72 rows=16500) (actual time=0.193..2.88 rows=16044 loops=1)
										-> Hash
											-> Covering index scan on f using idx_title  (cost=103 rows=1000) (actual time=0.0431..0.177 rows=1000 loops=1)
									-> Covering index lookup on r using rental_date (rental_date = p.payment_date)  (cost=0.594 rows=1.04) (actual time=923e-6..0.00123 rows=1.01 loops=634000)
								-> Single-row index lookup on c using PRIMARY (customer_id = r.customer_id)  (cost=250e-6 rows=1) (actual time=91e-6..105e-6 rows=1 loops=642000)
							-> Single-row covering index lookup on i using PRIMARY (inventory_id = r.inventory_id)  (cost=250e-6 rows=1) (actual time=83.1e-6..97.3e-6 rows=1 loops=642000)
		
```


Стало
---
```SQL
	-> Sort: full_name, c.last_name  (actual time=6.5..6.52 rows=391 loops=1)
		-> Table scan on <temporary>  (actual time=6.34..6.38 rows=391 loops=1)
			-> Aggregate using temporary table  (actual time=6.34..6.34 rows=391 loops=1)
				-> Nested loop inner join  (cost=4886 rows=1833) (actual time=0.245..5.78 rows=634 loops=1)
					-> Nested loop inner join  (cost=4244 rows=1833) (actual time=0.24..5.1 rows=634 loops=1)
						-> Nested loop inner join  (cost=3603 rows=1833) (actual time=0.236..4.65 rows=634 loops=1)
							-> Filter: ((p.payment_date >= TIMESTAMP'2005-07-30 00:00:00') and (p.payment_date < TIMESTAMP'2005-07-31 00:00:00') and (p.rental_id is not null))  (cost=1674 rows=1833) (actual time=0.224..3.86 rows=634 loops=1)
								-> Table scan on p  (cost=1674 rows=16500) (actual time=0.217..3.04 rows=16044 loops=1)
							-> Single-row index lookup on r using PRIMARY (rental_id = p.rental_id)  (cost=0.952 rows=1) (actual time=0.00113..0.00114 rows=1 loops=634)
						-> Single-row index lookup on c using PRIMARY (customer_id = r.customer_id)  (cost=0.25 rows=1) (actual time=615e-6..629e-6 rows=1 loops=634)
					-> Single-row covering index lookup on i using PRIMARY (inventory_id = r.inventory_id)  (cost=0.25 rows=1) (actual time=960e-6..974e-6 rows=1 loops=634)
```


## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

## Задание 3*
Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

Приведите ответ в свободной форме.