---
title: SQLのウィンドウ関数
name:  sql-window-functions
---

SQLのウィンドウ関数
================

_**ご注意**: この記事のSQLクエリーはPostgreSQLだけで動いています。あなとのRDBMSの文書を確認してください！_

しばらく前に私はウィンドウ関数を知りました。とても役に立つものになりましたから、 今日、データベースのレコードの再番号を付け替えることを例としてウィンドウ関数について話したいです。

このごろ、私は会社で開発していたアプリをマルチテナントのアプリに変造していました。アプリのユーザーはいろいろな「駅」で働いていますから。一つの駅のデータベースのレコードは他の駅のデータに影響しないはずです。外部キーの追加は簡単ですけど、存在のデータをどうしたらいいでしょう？

アプリで「運送状」というモデルがあります。これは自動車の旅程と目的を含まれている記録です。これまで運送状の順番番号として`id`は使われていました。今は順番番号は駅とシリーズの組み合わせ毎の中に又と無いはずです。

まず、運送状と駅を直接の関連を作らなければなりません。データ図を見ると、運送状は自動車に従属して、自動車は駅に従属しています。それでマイグレーションでは運送状のテーブルに`station_id`という外部キーカラムを追加して、それとも`vehicles`から`waybills`テーブルに駅の識別子をコピーしている簡単なクエリーを書きます。

```PostgreSQL
UPDATE waybills
SET station_id = vehicles.station_id
FROM vehicles
WHERE vehicles.id = waybills.vehicle_id
```

それから`number`というカラムを追加します。このカラムを個々別々の駅とシリーズにとって順番の番号で記入しなければなりません。ウィンドウ関数は手伝ってくれます！

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

この問い合わせは何をしているのでしょうか？

外部クゥエリでPostgreSQLだけの`UPDATE … FROM`構文を使っています。`number`というカラムを副問い合わせから戻った同じ`id`の行の値で記入しています。

副問い合わせでは、`waybills`というテーブルの毎の組の`id`を取りながら、`row_number()`というウィンドウ関数を使ってこの組のパーティションの中で組の順番番号を計算しています。

ウィンドウ関数の呼び出し方をご覧ください：`関数 OVER (パーティション)`。 パーティションは`PARTITION BY`句と表現リストで定義されています。私はこの表現としてただカラムの名前を使っています。 

`PARTITION BY`リストは、行を`PARTITION BY`式の同じ値を共有するグループに分割する指定を行います。 それぞれの行に対し、ウィンドウ関数は現在行と同じパーティションに分類される行に渡って計算されます。`GROUP BY`に対して、パーティションの行は一体にされないで、毎の行にとってウィンドウ関数は計算されています。`ORDER BY`句を使って行の順序の制御することができます。

そこで、全部の行を駅とシリーズで分割して、毎のグループの中で作成時間によってソートしてから（グループの一番古いレコードに`row_number()`は`1`を戻ります）、行の順番番号を計算しました。

この副問い合わせを模擬のデータで試しましょう:

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

すてき！

完成したマイグレーションのファイルはこのようになります：

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

結論
---

SQLはとても強い工具ですが、全部の能力が覚えられないのですから、このような記事が人に毎日使っている道具をよく分かることに手伝っていると思います。


もっと詳しい資料
-------------

 * [PostgreSQLのウィンドウ関数のチュートリアル（英語）](http://www.postgresql.org/docs/9.4/static/tutorial-window.html)
 * [PostgreSQLのウィンドウ関数のチュートリアル（日本語）](http://www.postgresql.jp/document/9.4/html/tutorial-window.html)
 * [PostgreSQLのウィンドウ関数のリスト（英語）](http://www.postgresql.org/docs/9.4/static/functions-window.html)
 * [PostgreSQLのウィンドウ関数のリスト（日本語）](http://www.postgresql.jp/document/9.4/html/functions-window.html)
