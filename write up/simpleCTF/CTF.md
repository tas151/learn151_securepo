#以下はsimple ctfを解くときの軌跡を記録する

**1　How many services are running under port 1000?**  
  ・ポート番号を調べるため、  
  nmap -sV IP　を試行。（バージョンまでの調査を癖づける）  
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

⇒上記結果から”2”と回答  
2 **What is running on the higher port?**  
⇒上記結果から”ssh”と回答  

3 **What's the CVE you're using against the application?**  
<ins>上記結果から調査順を1.ftp、2.http、3.sshとする。</ins>  
ftp IP ユーザ名をAnonymousで調査  
　⇒つながったがlsの結果が返ってこなかった。  
<details>
<summary>FTP調査</summary>

```text
ftp> ls
229 Entering Extended Passive Mode (|||49297|)
⇒ポート49297に接続できていないということが分かった。（原因はファイアウォールが閉じているからかも？EPSVという受け渡し法が壊れている可能性もあり）

&nbsp;⇒しかし侵入できているため、下記コマンドで対処

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

&nbsp;⇒ローカル上にtxtファイルを転送した。
&nbsp;&nbsp;⇒しかし、中身を見るに今問いに関係がなさそうのため、保留

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

4 What's the password?

⇒http調査の結果、今回の脆弱性が**SQL Injection**と判明した。  
  しかし文字数が4文字のため**SQLI**回答。

5 What's the password?

⇒脆弱性とPOCが判明したため、実際に動かしていく  
　が、この46635.pyファイルが2.x用であったため、これを3.x用にしなければならない

 ⇒しかし、kaliのOSの中核であるpythonを削除したり上書きすると壊れてしまう。
 　そのため、pyenv（pythonバージョンの共存）を環境で使えるようにする。（別の部屋を追加するイメージ）

・bashrcの内容にpyenvを使えるようPATHとかを追加していたがずっとエラーがはかれた  
⇒何度も/etc/skel/bashrcから初期化したがなおらず  
⇒そもそもkali linuxはbashシェルじゃなくてzshシェルを使っていることが分かった(echo $SHELLより)
そのためpyenvの設定をzshrcに記載した<details>
<summary>46635.pyを3.x用に変換する際にものすごく詰まったため、何をやったかを記載</summary>

```text
内容を完全に理解できているとはいいがたいが、
pyenvを利用して3.11.9のバージョンを共存させた
それをpython3 -m python3 -w 46635.py で3.11用に変換した
（python3はlib2to3がある親ディレクトリ）
そのままでは使えないため、手動で修正を試みたのだが、ここで詰まった
最終的な原因としてはインデントのずれ、だと思われる
:set expandtabや:retabを使ったりしてインデントをそろえた
が、いろいろと修正しているため絶対にこれ!とはいえない

注)　一時的に3.11.9を有効化しているため、今後使用する際には
　　　pyenv shell 3.11.9　を実行すること
```
</details>

