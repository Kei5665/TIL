## メモ
- IDが2の店舗に所属するスタッフって誰だっけ？
select * staff from staff where store_id = 2
Staff.where(store_id: 2)

- 映画「BLANKET BEVERLY」のDVD、いくつ在庫あるっけ？
A = select * from film where title = 'BLANKET BEVERLY'
select sum(inventory_id) from inventory where film_id = A.film_id 

SELECT COUNT(*) FROM `inventory` INNER JOIN `film` ON `inventory`.`film_id` = `film`.`film_id` WHERE `film`.`title` = 'BLANKET BEVERLY'

Film.joins(:inventories).where(title:"BLANKET BEVERLY").count

Inventory.joins(:film).where(film: {title:"BLANKET BEVERLY"} ).count

- 2005年に貸し出ししてるけどまだ返却されてない映画があるらしい。その映画の一覧を出して！
A = select * from rental WHERE DATE_FORMAT(rental_date, '%Y') = 2005 and return_date is null

B = select * from Inventory join rental on inventory.inventory_id = A.inventory_id

C = select * from film join inventory on film.film_id = inventory.film_id

select * from film

join inventory on film.id = inventory.film_id

join rental on inventory.id = rental.inventory_id

WHERE DATE_FORMAT(rental_date, '%Y') = 2005 and return_date is null

select film.film_id,film.title from film join inventory on film.film_id = inventory.film_id join rental on inventory.inventory_id = rental.inventory_id WHERE DATE_FORMAT(rental_date, '%Y') = 2005 and return_date is null

## activerecord
Film.distinct.joins(inventories: :rentals).select('film.title,film.film_id').where(rental: {return_date: [nil]}).where('rental.rental_date LIKE ?',"2005%")

- 応用課題の振り返り(タグの挙動)
Article.distinct.joins(:tags).where(article_tags: { tag_id: 1 })

```
SELECT DISTINCT "articles".* FROM "articles"

INNER JOIN "article_tags" ON "article_tags"."article_id" = "articles"."id" 

INNER JOIN "taxonomies" ON "taxonomies"."id" = "article_tags"."tag_id" AND "taxonomies"."type" IN ('Tag')

WHERE "article_tags"."tag_id" = ?  [["tag_id", 1]]
```

## コメント
nilを指定するときは[nil]にする

- ある問題
#### ざっくりな見通し
select sum(amount) from payment group by category

#### 問題点
paymentテーブルにcategoryカラムがないので、amountをcategoryごとに集計できない

#### 課題
amountごとのcategoryを把握したい

#### 現状
おそらく１回の会計につき１つの在庫が出ていく

#### ざっくりな見通しの再度確認
もしもpaymentテーブルにカテゴリーカラムがあったなら
select sum(payment.amount) from payment group by category

#### 実験
select category.category_id, category.name, sum(payment.amount) from payment INNER JOIN `rental` ON `payment`.`rental_id` = `rental`.`rental_id` INNER JOIN `inventory` ON `rental`.`inventory_id` = `inventory`.`inventory_id` INNER JOIN `film` ON `inventory`.`film_id` = `film`.`film_id` INNER JOIN `film_category` ON `film`.`film_id` = `film_category`.`film_id` INNER JOIN `category` ON `film_category`.`category_id` = `category`.`category_id` GROUP BY category.category_id, category.name ORDER BY sum(payment.amount) DESC LIMIT 5

#### activerecord
Payment.select("category.category_id, category.name, sum(payment.amount) as revenue").joins(rental: {inventory: {film: :categories} } ).group("category.category_id, category.name").limit(5).order(revenue: :desc)

#### コメント
まさかとは思ったが、結合するテーブルがこんなにも多いこともあるのかと勉強になった。
activerecordもネストしまくりだった。
自分が間違っているんじゃないかと不安になったが方向性は合っていたと思う。
テーブル結合をするとselect句やgroup by句で結合したテーブルのカラムを指定できることは勉強になった。
疑問はactiverecordの時、指定してないpaymentの主キーが出力された件について。主キーは省略できないものだろうか？

- ある問題
#### ざっくりな見通し
inventoryテーブルにcategoryカラムがあった場合
select sum(inventory_id) from inventory group by category

#### 問題点
inventoryテーブルにcategoryカラムがない

#### 課題
inventoryテーブルにcategoryテーブルを結合させ、categoryごとに合計を出す

#### 仮説
select category.category_id, category.name, count(inventory_id as number) from inventory

join film on inventory.film_id = film.film_id

join film_category on film.film_id = film_category.film_id

join category on film_category.category_id = category.category_id

group by category.category_id, category.name

#### 実験
select category.category_id, category.name, count(inventory_id) from inventory join film on inventory.film_id = film.film_id join film_category on film.film_id = film_category.film_id join category on film_category.category_id = category.category_id group by category.category_id, category.name

#### 気づき
どうやら「カテゴリ別に映画の本数を集計して」の本数というのは在庫が抱える映画の数ではなく
カテゴリーが持っている映画の数ということらしい

#### 修正したざっくりな見通し
film_categoryの総数をカテゴリーごとに集計すれば良い
select count(*) from film_category group by category.name

#### 問題点
film_categoryにcategory.nameカラムがない

#### 課題
film_categoryにcategoryカラムを結合させる

#### 仮説
select category.category_id, category.name, count(*) as number_of_film from film_category

join category on film_category.category_id = category.category_id

group by category.name

having count(*) >= 60

order by number_of_film desc

#### 実験
select category.category_id, category.name, count(*) as number_of_film from film_category join category on film_category.category_id = category.category_id group by category.category_id, category.name having count(*) >= 60 order by number_of_film desc

#### activerecord
FilmCategory.select("category.category_id, category.name, count(*) as number_of_film").joins(:category).group("category.category_id, category.name").having("number_of_film >= ?", 60).order(number_of_film: :desc)

- ある問題
#### ざくっりとした見通し
filmテーブルにfilm_actorとactorを結合させる
select film_id, title from film
join film_actor on film.film_id = film_actor.film_id
join actor on film_actor.actor_id = actor.actor_id
where actor.first_name = "JOE" and actor.last_name = "SWANK"

#### 実験
select film.film_id, film.title from film join film_actor on film.film_id = film_actor.film_id join actor on film_actor.actor_id = actor.actor_id where actor.first_name = "JOE" and actor.last_name = "SWANK"

#### activerecord
Film.select("film.film_id, film.title").joins(:actors).where('actor.first_name = "JOE" and actor.last_name = "SWANK"')

- ある問題
#### ざっくりとした見通し
もしfilmテーブルにactorカラムがあったなら
filmテーブルからタイトルが「JOE SWANK」で、lengthが100以上のものを抽出する

select film.film_id, film.title, film.length from film
join film_actor on film.film_id = film_actor.film_id
join actor on film_actor.actor_id = actor.actor_id
where actor.first_name = "JOE" and actor.last_name = "SWANK" and film.length >= 100

#### 仮説
select film.film_id, film.title, film.length from film join film_actor on film.film_id = film_actor.film_id join actor on film_actor.actor_id = actor.actor_id where actor.first_name = "JOE" and actor.last_name = "SWANK" and film.length >= 100

#### activerecord
Film.select("film.film_id, film.title, film.length").joins(:actors).where('actor.first_name = "JOE" and actor.last_name = "SWANK" and film.length >= 100')

#### コメント
activerecordでwhereは分けたほうが見やすいかもしれない

- ある問題
#### ざっくりとした見通し
もしfilmテーブルにcategoryカラム、actorカラムがあったなら
select film.film_id, film.title from film
join film_category on film.film_id = film_category.film_id
join film_actor on film.film_id = film_actor.film_id
join category on film_category.category_id = category.category_id
join actor on film_actor.actor_id = actor.actor_id
where actor.first_name = "JOE" and actor.last_name = "SWANK" and category.name = "Action"

#### 実験
select film.film_id, film.title from film join film_category on film.film_id = film_category.film_id join film_actor on film.film_id = film_actor.film_id join category on film_category.category_id = category.category_id join actor on film_actor.actor_id = actor.actor_id where actor.first_name = "JOE" and actor.last_name = "SWANK" and category.name = "Action"

#### activerecord
Film.select("film.film_id, film.title").joins(:categories, :actors).where('actor.first_name = "JOE" and actor.last_name = "SWANK" and category.name = "Action"')

- ある問題
#### ざっくりとした見通し
paymentテーブルの中で「JOE SWANK」の映画が借りられているレコードの、amountをtitleごとに集計する

paymentテーブルにactorカラム、映画のタイトルカラムがあるとして

select film.film_id, film.title, sum(payment.amount) as revenue from payment
JOIN rental ON payment.rental_id = rental.rental_id
JOIN inventory ON rental.inventory_id = inventory.inventory_id
join film on inventory.film_id = film.film_id
join film_actor on film.film_id = film_actor.film_id
join actor on film_actor.actor_id = actor.actor_id
where actor.first_name = "JOE" and actor.last_name = "SWANK"
group by film.title, film.film_id
order by revenue desc
limit 10

#### 実験
select film.film_id, film.title, sum(payment.amount) as revenue from payment join rental ON payment.rental_id = rental.rental_id join inventory ON rental.inventory_id = inventory.inventory_id join film on inventory.film_id = film.film_id join film_actor on film.film_id = film_actor.film_id join actor on film_actor.actor_id = actor.actor_id where actor.first_name = "JOE" and actor.last_name = "SWANK" group by film.title, film.film_id order by revenue desc limit 10

#### activerecord
Payment.select("film.film_id, film.title, sum(payment.amount) as revenue").joins(rental: {inventory: {film: :actors}}).where('actor.first_name = "JOE" and actor.last_name = "SWANK"').group("film.title, film.film_id").order(revenue: :desc).limit(10)

#### コメント
group by句を使う場合、select句で指定するカラムはgroup by句でも指定する。

- ある問題
#### ざっくりとした見通し
paymentテーブルにfilm_titleカラムがあった場合
select sum(payment.amount) from payment where film_title = "SUNRISE LEAGUE"

select film.film_id, film.title, sum(payment.amount) as revenue from payment
join rental ON payment.rental_id = rental.rental_id
join inventory ON rental.inventory_id = inventory.inventory_id
join film on inventory.film_id = film.film_id
where film.title = "SUNRISE LEAGUE"
group by film.title, film.film_id

#### 実験
select film.film_id, film.title, sum(payment.amount) as revenue from payment join rental ON payment.rental_id = rental.rental_id join inventory ON rental.inventory_id = inventory.inventory_id join film on inventory.film_id = film.film_id where film.title = "SUNRISE LEAGUE" group by film.title, film.film_id

#### activerecord
Payment.select("film.film_id, film.title, sum(payment.amount) as revenue").joins(rental: {inventory: :film}).where('film.title = "SUNRISE LEAGUE"').group("film.title, film.film_id")

#### コメント
sum()を使う時、select句で他のカラムも指定するときはgroup byを使うのかも