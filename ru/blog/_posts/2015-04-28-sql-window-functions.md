---
title: Оконные функции в SQL
name:  sql-window-functions
---

Оконные функции в SQL
=====================

_**Примечание**: Запросы в данной статье работают в PostgreSQL и могут не работать в вашей СУБД, сверяйтесь с документацией!_

Недавно я открыл для себя существование оконных функций в SQL. Расскажу о них на примере ситуации, когда они мне помогли: перенумерация записей в базе.

В то время я вносил мультитенантность в наше приложение — оно должно было работать в рамках разных станций. Соответственно, все сущности должны существовать в рамках станции и никогда не влиять на такие же сущности на других станциях. Добавить внешних ключей в таблицы — плёвое дело. Но что делать с существующими данными?

Итак, ситуация: у нас в приложении есть «путевые листы», они пронумерованы по порядку, в качестве номера используется просто ID. Но теперь оказывается, что должна быть своя нумерация в рамках каждой станции, а на станциях у документов есть ещё и серии, и внутри каждой серии тоже должна быть своя нумерация.

Сперва привяжем путевые листы напрямую к станциям. Смотрим схему данных: путевые листы привязаны к машинам, машины — к станциям, добавляем в миграции колонку `station_id` и пишем простой запрос, который добавит ID станции в таблицу `waybills`:

```PostgreSQL
UPDATE waybills
SET station_id = vehicles.station_id
FROM vehicles
WHERE vehicles.id = waybills.vehicle_id
```

Теперь надо заново перенумеровать все путевые листы в рамках станции и серии. Тут-то нам и помогут оконные функции:

```PostgreSQL
UPDATE waybills
SET number = w.number
FROM (
  SELECT
    id,
    row_number() OVER (
      PARTITION BY station_id, series ORDER BY created_at
    ) AS number
  FROM waybills
) w
WHERE waybills.id = w.id;
```

Итак, что же здесь происходит?

Во внешнем запросе мы используем уже виденный ранее синтаксис `UPDATE … FROM`, специфичный для PostgreSQL, мы заполняем колонку `number` значением, которое нам вычисляет подзапрос для записи с таким же `id`, как и в подзапросе.

В подзапросе же мы выбираем `id` каждой строки в нашей таблице `waybills` и с помощью оконной функции `row_number()` вычисляем порядковый номер этой строки в заданном разбиении.

Обратите внимание на синтаксис вызова оконных функций: `функция OVER (разбиение)`. Разбиение задаётся ключевым словом `PARTITION BY` со списком колонок, по уникальным значениям которых строки и разбиваются на отдельные группы. Отличие от `GROUP BY` как раз и состоит в том, что строки в группах не схлопываются в одну, а обрабатываются по отдельности. Ключевое слово `ORDER BY` позволяет упорядочить строки внутри каждой группы.

Таким образом мы разбили строки на группы для каждой станции и серии документа и строки в каждой группе отсортировали по старшинству — от старых к новым. (функция `row_number()` вернёт `1` для самой старой строки в группе). 

Пробуем подзапрос на тестовых данных:

    =# SELECT id, station_id, series, created_at, row_number() OVER (PARTITION BY station_id, series ORDER BY created_at) AS number FROM waybills;
     id |              station_id              | series |         created_at         | number
    ----+--------------------------------------+--------+----------------------------+--------
      5 | 258d39b6-7b09-49ad-94f6-33b0dcb2fa32 |        | 2015-04-07 10:53:44.736078 |      1
      1 | 69c5b783-034c-47ea-84fd-c4abd0a323d6 | 123    | 2015-03-16 09:55:16.689384 |      1
      6 | 69c5b783-034c-47ea-84fd-c4abd0a323d6 | 123    | 2015-04-28 13:47:16.397076 |      2
      7 | 69c5b783-034c-47ea-84fd-c4abd0a323d6 | 123    | 2015-04-28 13:47:40.23337  |      3
      2 | 69c5b783-034c-47ea-84fd-c4abd0a323d6 | 23     | 2015-03-18 12:06:25.688768 |      1
      4 | 69c5b783-034c-47ea-84fd-c4abd0a323d6 | 4606   | 2015-03-24 08:10:38.143429 |      1
      3 | 69c5b783-034c-47ea-84fd-c4abd0a323d6 | 656565 | 2015-03-18 15:30:10.709491 |      1
    (7 rows)

Отлично!

Миграция в полном виде будет выглядеть так:

```ruby
class AddMultistationSupportForWaybills < ActiveRecord::Migration
  def change
    change_table :waybills do |t|
      t.integer :number
      t.references :station, type: :uuid
    end
    add_foreign_key :waybills, :stations
    
    reversible do |to|
      to.up do
        execute <<-PostgreSQL.strip_heredoc.tr("\n", ' ')
          UPDATE waybills
          SET station_id = vehicles.station_id
          FROM vehicles
          WHERE vehicles.id = waybills.vehicle_id
        PostgreSQL
        execute <<-PostgreSQL.strip_heredoc.tr("\n", ' ')
          UPDATE waybills
          SET number = w.number
          FROM (
            SELECT row_number() OVER (PARTITION BY station_id, series ORDER BY created_at) AS number, id FROM waybills
          ) w
          WHERE waybills.id = w.id;
        PostgreSQL
      end
    end
  end
end
```

Заключение
----------

SQL обладает огромными возможностями на все случаи жизни. Но запомнить их все — пожалуй, непосильная задача. Надеюсь, что данная статья поможет людям изучить этот инструмент чуть лучше.


Материалы для изучения
----------------------

 * [Руководство по оконным функциям в документации PostgreSQL на английском](http://www.postgresql.org/docs/9.4/static/tutorial-window.html)
 * [Это же руководство на русском (для версии постарше)](http://postgresql.ru.net/manual/tutorial-window.html)
 * [Документация по доступным оконным функциям PostgreSQL](http://www.postgresql.org/docs/9.4/static/functions-window.html)
