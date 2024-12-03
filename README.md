### 12-05.md
### Домашнее задание к занятию «Индексы» - Семиошко Андрей

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Решение 1

```sql
select	ROUND(SUM(index_length)*100/SUM(data_length)) 'size index in %'
from	INFORMATION_SCHEMA.TABLES
```
### Задание 2

Выполните explain analyze следующего запроса:

```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Решение 2

- перечислите узкие места:

Узкие места наблюдаются в момент использования оконных функций OVER и PARTITION BY, а  так же в  момент фильтрации вывода.

- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы:

Добавим индекс по столбцу payment_date таблицы payment
```sql
CREATE index payment_date on payment(payment_date);
```
и выполним оптимизированный запрос

```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) 
from payment p
join rental r on r.rental_id = p.rental_id 
join customer c ON c.customer_id = p.customer_id 
join inventory i on i.inventory_id = r.inventory_id 
where date(p.payment_date) = '2005-07-30' 
group by concat(c.last_name, ' ', c.first_name); 
```
![image](https://github.com/semioshkoan/12-05.md/blob/main/%D0%92%D1%8B%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5_097.png)
