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
**What is running on the higher port?**  
⇒上記結果から”ssh”と回答  

**What's the CVE you're using against the application?**  
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
　⇒
