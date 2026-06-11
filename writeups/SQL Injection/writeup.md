## SQL Injection（TryHackMe）
SQL Injection は、Webアプリが SQL を文字列として組み立てているため、
入力値がそのまま SQL 文に混ざってしまう脆弱性です。
SQL を使うリレーショナルデータベース特有の問題で、
' OR '1'='1 のように条件を改ざんされたり、
ブラインドSQLiのように情報を少しずつ抜かれることがあります。
対策はプレースホルダを使って、SQL と入力値を分離することです。

# 学んだこと
・SQL = Structured Query Language(構造化問い合わせ)
・DBMS：データベースを制御するソフトウェアの総称

SQLにはリレーショナル型とノンリレーショナル型の2つがある　
リレーショナル型：Excelみたいに列と行があって、きれいに並んでいるもの  
ノンリレーショナル型：メモアプリみたいに、何かいてもいい白紙のようなもの。ルールが緩い
　⇒SQLiに関係するのは**リレーショナル型だけ**（SQLを使うのはリレーショナルだけだから、ノンリレーショナルはSQLiは起きない（別の攻撃はある））

 RDB（リレーショナルデータベース）と呼ばれるもの、以下が代表例⇒RDBはテーブル同士をくっつけて、1つの結果として扱えるデータベース（この操作をJOINという）
 ・MySQL
 ・Microsoft
 ・SQLServer
 ・Access
 ・PostgreSQL
 ・SQLite

 キーフィールド：テーブルの中で1行を一意に特定できる列

 ・NoSQL＝Excelみたいな表を使わずに、メモアプリのように自由な形でデータを保存できるデータベース。

 SELECTクエリ：取得コマンド　⇒　ここでSQLiが起こる
 LIMIT 1：一行抽出　LIMIT 1,1 ：2行目抽出　　LIMIT 2,1：3行目抽出・・・
 where xxx= 'aaa' ：aaaと完全一致する行xxxの抽出
 where xxx!= 'aaa'：aaa以外のxxxのすべて抽出
 where xxx= 'aaa' or yyy= 'bbb'：aaaまたはbbbのxxxもしくはyyyの抽出（どちらか一つでも当てはまっていればOK）
 where xxx='aaa' and yyy='bbb'：xxxがaaaであり、yyyがbbbである行の抽出
 where xxx like '%aa%'：xxxの中にaaの文字を含む行抽出


UNION：同じ形の表どうしを上からくっつけて、1つの表にする機能
