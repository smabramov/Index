# «Индексы»-Абрамов Сергей

---

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

## Решение

```
SELECT round (sum(index_length) / sum(data_length) * 100) AS '% Index'
FROM INFORMATION_SCHEMA.TABLES;

```
![]()

---


### Задание 2

Выполните explain analyze следующего запроса:

```
EXPLAIN ANALIZE
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id

```

![]()

Колличество циклов 642000, затраченное время 4821 ms

## Решение

Запрос выдает оплату клиентов за определенную дату. Для данного анализа оставил таблицы payment и customer.


```
EXPLAIN ANALIZE
SELECT distinct concat(c.last_name, ' ', c.first_name) as Клиент, sum(p.amount) over (partition by c.customer_id) as 'Оплата'
from payment p, customer c
where date(p.payment_date) = '2005-07-30' and p.customer_id = c.customer_id;

```

![]()

В результате колличество циклов сократилось до 634 и затраченное время составило 5.94 ms. 

```
CREATE index day_payment on payment(payment_date);

```
Создан индекс даты платежа.


![]()

Переделан запрос с объединением таблиц.

```
EXPLAIN ANALIZE
SELECT concat(c.last_name, ' ', c.first_name) AS Клиент, SUM(p.amount) as Платеж
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id 
JOIN payment p ON r.rental_date = p.payment_date 
join inventory i on i.inventory_id = r.inventory_id 
where date(p.payment_date) >= '2005-07-30' and date(p.payment_date) < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY c.customer_id;

```

![]()


В результате колличество циклов  16044 и затраченное время составило 0.493 ms. 


