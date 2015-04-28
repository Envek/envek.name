---
title: SQL window functions
name:  sql-window-functions
---

SQL window functions
====================

_**Attention**: Queries in this article may work only in PostgreSQL, check docs for your RDBMS!_

Some time ago I have discovered window functions for myself. I will talk about them on example, when they helped me: database record renumbering.

That time I have been adding a multitenancy in our application — users in it should work in scope of different stations. All the entities in one station should not affect any entities on another. Adding foreign keys to tables easy. But what about existing data?

So, in our app there is a “Waybill” model. This is information about document that reflects a vehicle route and job. Waybills are numbered and at the time ID was used as a number. But afterwards waybill numbering should be unique in scope of station and document series (that was added at some point of time). 

First, linking waybills directly to stations. Looking at the data diagram I see: waybills belongs to vehicles, vehicles belongs to stations. So, in migration we will add foreign key column `station_id` and write SQL-query that will copy station identifiers from `vehicles` to `waybills` table:

```PostgreSQL
UPDATE waybills
SET station_id = vehicles.station_id
FROM vehicles
WHERE vehicles.id = waybills.vehicle_id
```

Then we will add column called `number` and now we need to fill it with ordered numbers for each station and series. Window functions to the rescue!

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

Well, what is going on there?

In outer query we are using PostgreSQL-specific `UPDATE … FROM` syntax. We’re filling `number` column with value returned from subquery for row with the same `id`.

In subquery we’re taking `id` of every row in our `waybills` table and calculating sequence number in each partition with `row_number()` window function.

Take a look to the window functions call syntax: `function OVER (partition)`. Partition is defined with `PARTITION BY` keyword with expressions list (we just use column names) or there can be no partition at all. Rows will be partitioned in groups by the all unique values of `PARTITION BY` expressions. Unlike `GROUP BY` rows in partition would not be merged in one row, but will be processed in sequence. `ORDER BY` keyword allows to reorder rows inside every partition.

So, we have partitioned all rows in groups for every station and every series in station and sorted rows from oldest to newest inside every group (function `row_number()` will return `1` for oldest row in group). 

Let's try subquery on test data:

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

Awesome!

Complete migration will look like this:

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

Conclusion
----------

SQL is very powerful and provides tools for almost every case. Remember all abilities is practically impossible. Hope that this article will help people to learn a bit more about excellent tools.


Learn more
----------

 * [Window functions tutorial in PostgreSQL docs](http://www.postgresql.org/docs/9.4/static/tutorial-window.html)
 * [List of available window functions in PostgreSQL](http://www.postgresql.org/docs/9.4/static/functions-window.html)
