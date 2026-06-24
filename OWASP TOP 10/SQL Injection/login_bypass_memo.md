usernameに ' OR 1=1 --しても、脆弱性をつけれない理由

SELECT * FROM users
WHERE username='<入力>'
AND password='<入力>';

'が2つなら値の外判定になってpasswordまでコメントアウトするだろうけど、'が3つで'の中に--が入っているため値の中判定されてpasswordは通常通りの判定になるということ
