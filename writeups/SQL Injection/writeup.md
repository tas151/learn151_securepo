## SQL Injection（TryHackMe）
SQLi は、ユーザー入力がそのままSQLに入ってしまい、
攻撃者がSQLを自由に書き換えられる脆弱性です。
例えば OR 1=1 を入れると条件が常に真になり、
本来見えないデータまで返ってしまいます。

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
INSERT：テーブルに新しい行挿入  
UPDATE：既存の行を上書きする  
DELETE：1行以上の削除。WHERE で“どの行を消すか”を決める。LIMIT で削除数を絞れる（MySQL系）

MySQL系
MySQLとその派生であるMairiaDBなど
MySQLとほぼ同じ文法で動くデータベース群をさす


# SQLインジェクションの流れ
普通のアクセス　
https://website.thm/blog?id=1　
アプリが内部で作るSQL：　
SELECT * FROM blog WHERE id=1 AND private=0 LIMIT 1;　
・id=1 の記事　(公開記事Aの指定)
・private=0（公開記事Aの指定)　
・1件だけ返す  
→ 普通のユーザーが見るときの動き  

id=2 は非公開なので普通は見れない　
URL：https://website.thm/blog?id=2　
内部SQL：SELECT * FROM blog WHERE id=2 AND private=0 LIMIT 1;　
id=2 は private=1（非公開）だから → 何も返らない（見れない）

攻撃者が細工したURLを使う  
攻撃者がこうアクセスする： https://website.thm/blog?id=2;--  
⇒ SQLi のスタート地点

SELECT * FROM blog WHERE id=2;-- AND private=0 LIMIT 1;
; → SQL文を終わらせる
-- → その後ろを全部コメントアウトする
;-- → --(コメントアウト)の無効化
　⇒AND private=0 LIMIT 1 が無効化される
   ⇒結果、非公開記事(id=2)が返ってくる。LIMIT 1も無効化されるから非公開記事すべて、となる。
（AND private=0は公開記事だけ返すように条件付けしている）

実際に実行されるSQL  
SELECT * FROM blog WHERE id=2; → private=0 のチェックが消えた！

結果：非公開記事が見えてしまう  
本来は見れないはずの「id=2（非公開）」の記事が返ってくる。
これが SQLインジェクション（インバンド型）。

普通に id=2 を指定しても見れない。でも ;-- を入れて SQL の後半を消すと、
“公開記事だけ”という条件が消えて、非公開記事まで見えてしまう。




