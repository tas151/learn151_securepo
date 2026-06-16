# Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

## 概要
このラボでは、"category="にwhere句が仕込まれておりサニタイズしていなかったため、SQLインジェクションが実現した。
意図しない動き（商品の全表示）を実現した。

## 対象エンドポイント
”category=”が攻撃委ポイントとなっている。
”category=' OR 1=1 --”で実現。

## 脆弱性の原因
SQL文がURLに埋め込まれていた。
'や/などの文字に対してエスケープ処理がされていない。
