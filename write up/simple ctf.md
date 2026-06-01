#以下はsimple ctfを解くときの軌跡を記録する

1　How many services are running under port 1000?
  ⇒ポート番号を調べるため、
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

    ⇒上記結果から”2”と回答
      ⇒上記結果から調査順を第一ftp、第二をhttp、第三をsshとする
        ⇒ftp IP でユーザ名Anonymousで入れないかを試行
          ⇒結果、
