# Comparison Query Operators for PostgreSQL

## where句で使える演算子
### 準備
テスト用のデータを用意する。

```
$ psql
=# CREATE DATABASE test;
=# \connect test;
=# CREATE TABLE test(name varchar(10), id integer);
```

これでtestと言う名前のデータベースの中に同じくtestと言う名前のテーブルが作成できた。  
作成したテーブルを確認するには、

```
=# \dt
```

と打ち込めば良い。無事作成出来ていれば、

```
List of relations
Schema | Name | Type  |  Owner  
--------+------+-------+---------
public | test | table | vagrant
(1 row)
```

と表示される。  
なお、テーブルのカラム要素が含まれているか知りたい場合は、

```
=# \d test;
```

と入力すれば良い。  
例えば、上で用意したようなテーブルでは以下のような出力が得られる。

```
Table "public.test"
Column |         Type          | Modifiers
--------+-----------------------+-----------
name   | character varying(10) |
id     | integer               |
```

以下で実際にテスト用のデータを挿入する。  

```
=# INSERT INTO test (name, id) VALUES
    ('tom',1),
    ('jack', 2),
    ('john', 3),
    ('mike', 4);
```

以下のselectクエリを発行してデータが挿入できているか確認出来る。

```
=# select * from test;
```
```
 name | id
------+----
 tom  |  1
 jack |  2
 john |  3
 mike |  4
(4 rows)
```

以下の節では例に挙げていくクエリは全て
```
SELECT * FROM test;
```
を基本形として使う。SELECTの直後の **\*** は全象記号。

### ORDER BY
postgreSQLではそのままORDER BYをSELECTクエリのWHERE節に追加する。
```
SELECT * FROM test ORDER BY [対象カラム名];  
```

ORDER BY後に指定する[対象カラム]で、どのカラムについて昇順/降順で出力するかを指定する。  
昇順か降順かを指定する方法は、カラム名の先頭に-(マイナス)を付けるか付けないかで指定できる。  
例えば、idについて昇順でカラムnameとカラムidを出力したければ、

```
=# SELECT * FROM test ORDER BY id;
```

と言うクエリを発行すれば、以下のようにレコードが返ってくる。

```
 name | id
------+----
 tom  |  1
 jack |  2
 john |  3
 mike |  4
(4 rows)
```

同じように、idについて降順にレコードを出力したければ、

```
=# select * from test order by -id;
```

以上のようなクエリを発行すれば目的の結果が得られる。

```
 name | id
------+----
 mike |  4
 john |  3
 jack |  2
 tom  |  1
(4 rows)
```

### LIMITとOFFSET
#### LIMIT
postgreSQLでは、以下のような構文が利用できる。
```
SELECT [selectカラム1], [selectカラム2] from [テーブル名] LIMIT n
```

LIMITの後のnには整数値が入る。  
selectによって取得できたレコード数がたとえnを超える場合でも、出力されるレコードの数(表示されるレコードの数)はn以上にはならない。  
例えば、以下のようなクエリを発行すると、  

```
=# SELECT * FROM test LIMIT 2;
```

ここではLIMITで2を指定しているので、通常なら4件出力されるようなクエリでも最初の二件のみ出力される。

```
name | id
------+----
tom  |  1
jack |  2
(2 rows)
```

#### OFFSET
postgreSQLでは、以下のような構文になる。
```
SELECT [selectカラム1], [selectカラム2], ... FROM [テーブル名] OFFSET n
```

OFFSET後にあるnには整数が入る。ここで指定したnの値は、返ってくるレコードの開始位置を表しており、期待される出力のうち、最初のレコードから飛ばす行数を指定する。  
例えば、以下のクエリを発行すると、

```
SELECT * FROM test OFFSET 3;
```

ここではOFFSETにて3と言う値を指定しているので、返されるレコードのうち最初の3件が飛ばされて、
結果的に以下のようなレコードが返されることになる。

```
name | id
------+----
mike |  4
(1 row)
```

LIMITとOFFSETをSELECTクエリ内で同時に指定することによって、複数あるレコードのうち中間のデータのみを取り出すことが可能になっている。  
以下のクエリで例を示す。

```
=# SELECT * FROM test OFFSET 1 LIMIT 2;
```

このクエリでは、SELECT \* クエリを飛ばしているので、
全部で4件の出力が得られることが予想されるが、  
まずその後のOFFSETで1を指定することによって、最初の1件のレコードが飛ばされる。  
更にその後のLIMITで2を指定することによって、出力を2件に制限することによって結果的にidカラム結果的に以下のレコードが帰ってくることになる。

```
name | id
------+----
jack |  2
john |  3
(2 rows)
```

### WHERE節
SELECTでWHERE節を利用することで、比較演算子を用いて複雑な条件を満たすようなレコードを取得することが出来る。
```
SELECT * FROM test WHERE [条件式]
```

以下の節では条件式に用いることの出来る句を示す。

#### NOT
```
SELECT * FROM test WHERE NOT [句];
```
NOTは他の句と組み合わせて使うことが多く、[句]の部分には以下の節で示す

* NULL
* IN
* BETWEEN

などで、そのまま否定の意味で用いられる。

#### NULL
```
SELECT * FROM test WHERE [カラム名] IS NULL
```

[カラム名]にはNULLの判定をしたいカラム名が入る。  
今回用意したデータにはNULLは使われて無いので、新しくNULLを持つようなレコードを挿入する。

```
=# INSERT INTO test (name, id) VALUES (NULL, 5);
```

以上のクエリを発行した上で、例えば以下のようなクエリを発行すると、

```
=# SELECT * FROM test WHERE name IS NULL;
```

以下のようなレコードが取得できる。

```
name | id
------+----
     |  5
(1 row)
```

また、NULLについても前節で挙げた **NOT** を用いることが出来て、

```
=# SELECT * FROM test WHERE name IS NOT NULL;
```

と言うクエリを発行すれば、nameカラムがNULL以外のものを取得できる。

```
name | id
------+----
tom  |  1
jack |  2
john |  3
mike |  4
(4 rows)
```

#### BETWEEN (比較演算子を用いた場合)
この節ではWHERE節で比較演算子を用いた場合のBETWEENの挙動について示す。
```
SELECT * FROM test WHERE [条件式];
```

[条件式]には、比較演算子

* \>
* \<
* \>=
* \<=

と論理演算子ANDもしくはORを用いた条件式を書く。  
例えば、以下のようなSELECTクエリを発行すると、

```
=# SELECT * FROM test WHERE 2 < id AND id <= 4;
```

指定した条件式では、カラム要素idが2を超える値"かつ"4以下となるような条件となっているので、
以下のようなレコードが返される。  

```
name | id
------+----
john |  3
mike |  4
(2 rows)
```

さらに、ORを用いて、以下のようなクエリを発行すると、  

```
SELECT * FROM test WHERE 2 > id OR id > 3;
```

条件式の部分ではidが2未満の値、[もしくは]3を超える値となるような条件になっているので、  
このクエリでは以下のようなレコードが得られる。

```
name | id
------+----
tom  |  1
mike |  4
(2 rows)
```

#### BETWEEN (BETWEEN構文を用いた場合)
また、同じくWHERE節でBETWEEN構文を使っても同じような操作が可能になっている。  
この構文の簡単な仕様は以下になる。
```
SELECT * FROM test WHERE [対象カラム名] [包含関係] BETWEEN m AND n;
```

クエリ内の[包含関係]には、そのまま何も指定しないか **NOT** の文字が入ることになる。  
何も指定しなかった場合は、

```
m <= [対象カラム名] <= n
```

の条件を満たすようなselectカラムが出力される。  
NOTが入った場合、条件式の不等号が反転され、包含関係が逆の出力がされることになる。

```
[対象カラム] < m, n < [対象カラム]
```

の条件式を満たすようなselectカラムが出力される。  
例えば、以下のようなクエリを発行すると、

```
=# SELECT * FROM test WHERE id BETWEEN 2 AND 4;
```

idが2以上かつ4以下と言う条件を指定したことになるので、以下のような出力が得られる。

```
name | id
------+----
jack |  2
john |  3
mike |  4
(3 rows)
```

また、以下のようなクエリを発行すると、

```
=# SELECT * FROM test WHERE id NOT BETWEEN 2 AND 3;
```

idの値が2未満かつ3を超える値のような条件を指定したことになるので、以下のような出力が得られる。

```
 name | id
------+----
 tom  |  1
 mike |  4
(2 rows)
```

#### IN
postgreSQLでは、 SELECTクエリのWHERE節のINで複数の条件を指定する。
```
SELECT * FROM test WHERE [対象カラム名] IN ([値1], [値2], ...);
```

[対象カラム名]にはIN選択のしたいカラム名を入れる。  
IN内部の[値1]、[値2]ではその対象カラムが取りうる目的の値を入力する。  
例えば、以下のクエリを発行すると、  

```
=# SELECT * FROM test WHERE name IN('john', 'mike');
```

ここではカラムnameがjohnとmikeとなっているレコードを取得することになるので、

```
name | id
------+----
john |  3
mike |  4
(2 rows)
```

以上のようなレコードが取得できる。

#### LIKE
```
SELECT * FROM test WHERE [カラム名] LIKE [検索文字];
```

[カラム名]には対象としたいカラムを書く。[検索文字]には、postgreSQL特有のパターンマッチメタ文字を使う。以下によく使うメタ文字を示す。

* **%** および **_**
それぞれ任意の文字列および任意の単一文字

* **\***
直前の項目の0回以上の繰り返しを意味する。

* **+**
直前の項目の1回以上の繰り返しを意味する。

* **?**
直前の項目の0回もしくは1回の繰り返しを意味する。

以上のメタ文字を使って、以下のLIKEを用いたクエリを発行すると、

```
=# SELECT * FROM test WHERE name LIKE 'j%';
```

以下のようなレコードが取得できる。

```
 name | id
------+----
 jack |  2
 john |  3
(2 rows)
```

また、検索対象となる文字の中に **%** などのメタ文字として登録されている記号が入っていたとしても、エスケープを用いることで検索の対象にすることが出来る。エスケープを用いる場合のLIKEクエリの書き方は以下になる。  
```
SELECT * FROM test WHERE [カラム名] LIKE [検索文字] ESCAPE [エスケープ文字];
```

エスケープ文字には新しいエスケープ文字を指定することが出来る。
動作を確認するために、再び新しくデータを挿入する。

```
=# INSERT INTO test (name, id) VALUES ('100%', 6);
```

この時、以下のクエリを発行すると、

```
=# SELECT * FROM test WHERE name LIKE '%#%' ESCAPE '#';
```

%が新しく定義したエスケープ文字 **#** によってエスケープ
されているのでname内に%を含むレコードも取得できる。

```
name | id
------+----
100% |  6
(1 row)
```

もしESCAPE句を利用したくない場合、**\\** がエスケープ文字として扱われる。

```
=# SELECT * FROM test WHERE name LIKE '%\%';
```

```
 name | id
------+----
 100% |  6
(1 row)
```

上でESCAPSE句を使った場合と同じレコードが取得できていることがわかる。

### 参考URL
https://www.postgresql.jp/document/9.4/html/index.html
