+++
date = "2017-05-01T20:20:57+09:00"
title = "MySQL 5.7系のGenerated_columnをRailsで使う"
description = "Generated_columnを試した話"
tags = ["MySQL", "Rails"]

+++

Generated_columnは、他のカラムの値を使って、仮想的なフィールドを設ける機能です。今回試したMySQLのバージョンは、`5.7.17` です。どんなものかは、コードを見た方が早いでしょう。

### Generated_columnを試す

テスト用にデータベースを作成し、商品テーブル（id, 名前, 単価, 数量, 価格）を作ります。Generated_columnは、合計値を求めたtotal_priceとなります。

```sql
CREATE DATABASE my_test;
CREATE TABLE my_test.products (
  id          INT(11)      NOT NULL PRIMARY KEY AUTO_INCREMENT,
  name        VARCHAR(255) NOT NULL,
  price       INT(11)      NOT NULL,
  unit        INT(11)      NOT NULL DEFAULT 0,
  total_price INT(11) AS (price * unit)
);
```

```
mysql> SHOW COLUMNS FROM my_test.products;
+-------------+--------------+------+-----+---------+-------------------+
| Field       | Type         | Null | Key | Default | Extra             |
+-------------+--------------+------+-----+---------+-------------------+
| id          | int(11)      | NO   | PRI | NULL    | auto_increment    |
| name        | varchar(255) | NO   |     | NULL    |                   |
| price       | int(11)      | NO   |     | NULL    |                   |
| unit        | int(11)      | NO   |     | 0       |                   |
| total_price | int(11)      | YES  |     | NULL    | VIRTUAL GENERATED |
+-------------+--------------+------+-----+---------+-------------------+
5 rows in set (0.00 sec)
```

データをInsertしてみます。

```sql
INSERT INTO my_test.products (name, price, unit) VALUES ("banana", 200, 3);
INSERT INTO my_test.products (name, price, unit) VALUES ("banana", 500, 10);
INSERT INTO my_test.products (name, price, unit) VALUES ("banana", 1500, 2);
INSERT INTO my_test.products (name, price, unit) VALUES ("apple", 20, 6);
INSERT INTO my_test.products (name, price, unit) VALUES ("apple", 250, 4);
INSERT INTO my_test.products (name, price, unit) VALUES ("apple", 150, 3);
```

照会してみると、合計値が算出されています。

```
mysql> SELECT * FROM my_test.products;
+----+--------+-------+------+-------------+
| id | name   | price | unit | total_price |
+----+--------+-------+------+-------------+
|  1 | banana |   200 |    3 |         600 |
|  2 | banana |   500 |   10 |        5000 |
|  3 | banana |  1500 |    2 |        3000 |
|  4 | apple  |    20 |    6 |         120 |
|  5 | apple  |   250 |    4 |        1000 |
|  6 | apple  |   150 |    3 |         450 |
+----+--------+-------+------+-------------+
```

更新すると、合計値も変わります。

```
mysql> UPDATE my_test.products SET price=250 WHERE id=1;

mysql> SELECT * FROM my_test.products WHERE id=1;
+----+--------+-------+------+-------------+
| id | name   | price | unit | total_price |
+----+--------+-------+------+-------------+
|  1 | banana |   250 |    3 |         750 |
+----+--------+-------+------+-------------+
```

作成済のテーブルにGenerated_columnを追加するときは、 `ALTER TABLE my_test.products ADD COLUMN foobar AS ()` といったALTER TABLE文を実行すれば問題ありません。見慣れないのは、`AS` だけですね。

当たり前ですが、集計も可能です。

```
mysql> SELECT name, SUM(total_price) FROM my_test.products GROUP BY name;
+--------+------------------+
| name   | SUM(total_price) |
+--------+------------------+
| apple  |             1570 |
| banana |             8750 |
+--------+------------------+
```

### Railsで試す

Rails5.1の場合は、[kamipoさんのコミット](https://github.com/rails/rails/commit/65bf1c60053e727835e06392d27a2fb49665484c)にある通り、virtualメソッドで定義します。

```ruby
create_table :generated_columns do |t|
  t.string  :name
  t.virtual :upper_name, type: :string,  as: "UPPER(name)"
  t.virtual :name_length, type: :integer, as: "LENGTH(name)", stored: true
  t.index :name_length  # May be indexed, too!
end
```

同じことをRails 5.0系以前で利用する場合は、executeを利用して、upとdownを定義します。reversibleなども使えますが、単純にup/downが分かりやすそうです。

```ruby
class AddGeneratedColumnForDate < ActiveRecord::Migration[5.0]
  def up
    execute 'ALTER TABLE products ADD COLUMN published_on DATE AS (DATE(published_at + INTERVAL 9 HOUR))'
  end

  def down
    remove_column :products, :published_on
  end
end
```

以降は通常通り、マイグレーションを実行すれば、Generated columnが定義されます。

### 注意点

利用してみて、挙動として注意したいのは以下です。

1つ目は、createした後に、generated_columnの値を参照しようとすると、その時点ではNULLであること（reloadする必要がある）こと。2つ目は、[zdennis/activerecord-import](https://github.com/zdennis/activerecord-import)を使う場合、ビルドしたインスタンスのコレクションを基に、import処理を行おうとすると、generated_columnに値をインサートしようとすると、MySQLのエラーとなることです。

以下のように、importで利用するカラムを明示的に定義すれば、これらは回避できます。

```ruby
books = [
  Book.new(:title => "Book 1", :author => "FooManChu"),
  Book.new(:title => "Book 2", :author => "Bob Jones"),
  Book.new(:title => "Book 1", :author => "John Doe"),
  Book.new(:title => "Book 2", :author => "Richard Wright")
]
columns = [ :title ]

# 2 INSERT statements for 4 records
Book.import columns, books, :batch_size => 2
```
