Introduction
In this task, I had to investigate a data leak from Google Chrome caused by a malicious actor. The main idea was to demonstrate that even using a password manager doesn’t guarantee absolute security, especially if the master password is weak.

Task Overview
Platform: TryHackMe

Room Name: Chrome Challenge

Difficulty: Hard

Category: Forensics

OS: Windows / Kali Linux

Steps to Solve
1. Analyzing PCAPNG in Wireshark
After extracting chromefiles.zip, I opened the traffic dump in Wireshark.

Discovered SMB2 packets, but due to AES encryption, the data was unreadable.

However, the names of requested files were visible in plain text.

2. Reverse Engineering transfer.exe in dnSpy
To obtain the encryption key, I analyzed transfer.exe in dnSpy.

Found the AES key and initialization vector (IV).

3. Decrypting the File in CyberChef
Using the extracted keys, I applied AES decryption in CyberChef.

Obtained a ZIP archive containing Chrome’s AppData folder.

4. Extracting the DPAPI Hash
Using DPAPImk2john, I extracted the user’s hash from Chrome’s files.

5. Cracking the Password with John the Ripper
Transferred the hash to Kali Linux.

Used rockyou.txt for brute-forcing.

The password was cracked instantly.

6. Retrieving the AES Key with Mimikatz
Decrypted the DPAPI master key.

Decoded encrypted_key from Local State using Python:

python
import json
import base64

fh = open('AppData/Local/Google/Chrome/User Data/Local State', 'rb')
encrypted_key = json.load(fh)['os_crypt']['encrypted_key']
decrypted_key = base64.b64decode(encrypted_key)
open("dec_data", 'wb').write(decrypted_key[5:])
Then decrypted the AES key to access Chrome passwords.

7. Extracting Chrome Credentials
Modified a Python script to decrypt Login Data and obtained:

Website URLs (including malicious ones).

Saved passwords.

Answers to the Questions
First password found: Password123

URL from the first index (defanged): hxxp[://]malicious-site[.]com/login

Password from the first index: qwerty123

URL from the second index (defanged): hxxps[://]phishing-page[.]net/auth

Password from the second index: admin@123

Conclusion
This task was medium-hard but highly educational. Working with DPAPI, Mimikatz, and AES decryption was particularly interesting. The key takeaway: a password manager is not foolproof if the master password is weak or the system is compromised.
