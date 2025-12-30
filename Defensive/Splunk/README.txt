--Setup--
	--Download
		I will be using Windows 11 for this. Download Splunk at https://www.splunk.com/en_us/download/splunk-enterprise.html?locale=en_us.
		The download setup is very simple, so I will not include instructions.
		Once it's downloaded, it is hosted on localhost:8000, http://127.0.0.1:8000.
		I will be using the BOTS dataset, https://github.com/splunk/botsv3?tab=readme-ov-file.
		BOTS is a CTF-style challenge specifically for Splunk, but I will only be using it for the data, not the CTF challenges.
		
	--Data
		You have to import the data as an app.
		Click "Apps" in the top left and then click "Manage Apps."
		Install app from file, and then select the .tgz compressed folder.
		Then restart it by going to Settings -> Server controls -> Restart Splunk.
		
	--Verify Data
		Under "Apps", click "Search & Reporting", then search for "index=botsv3 earliest=0".
		This is a massive data set(2.1 million events), so this is just to confirm it exists.
    I will only be using Windows security event logs.

--Credential Abuse--
	4648 - Explicit Credential Use
		--Use & Limitations
      Using the statistics tab, look for human accounts, human logons should not explicitly use credentials
      So this can be a sign of an attacker using valid credentials
      Expects 2 Account_Name variables, with the second one being the target
      Filters out specific system accounts
				
    --Query
      index=botsv3 sourcetype=WinEventLog:Security EventCode=4648
      | eval target = mvindex(Account_Name,1)
      | where NOT target IN ("DWM-1", "UMFD-0", "UMFD-1")
      | stats count by target, ComputerName
      
  5059 - Cryptographic Key Export/Import
    --Use & Limitations
      This query shows user export key requests, which is suspicious since users shouldn't need to export keys
      In the statistics tab, look for suspicious entries
      It is already set up to filter service accounts, and the operation is limited to export. This gets rid of valid entries like key rotation
      The viewer just has to manually inspect/correlate the remaining user export requests
    
    --Query
      index=botsv3 sourcetype=WinEventLog:Security EventCode=5059
      | where Operation LIKE "Export%"
        AND NOT Account_Name LIKE "%$" 
        AND NOT Account_Name IN ("LOCAL SERVICE")
      | table _time Account_Name Operation Key_Name ComputerName
      
      
Priv. Esc.
  4672 - Privileged Logon
    --Use & Limitations
      Shows how many times a user(target) logs in with privileges vs non-privileged logins
      A target with a ratio that isn't 1 or 0 is worth investigating
      Be careful of rounding. If a user has 1000 normal logins but 1 privileged login, it would show up as a ratio of 0
      When in reality that is the exact scenario that is the most suspicious, you can adjust the decimal points if needed
      Filters out specific system accounts
    
    --Query
      index=botsv3 sourcetype=WinEventLog:Security (EventCode=4624 OR EventCode=4672)
      | where NOT Account_Name IN ("SYSTEM", "LOCAL SERVICE", "NETWORK SERVICE", "DWM-1", "UMFD-0", "UMFD-1")
      | eval logon=if(EventCode=4624, 1, 0)
      | eval privileged=if(EventCode=4672, 1, 0)
      | eval target = if(EventCode==4624, mvindex(Account_Name,1), Account_Name)
      | stats 
        sum(logon) as totalLogons
        sum(privileged) as privilegedLogons
        by target
      | eval normalLogons = totalLogons - privilegedLogons
      | eval privilegeRatio = round(privilegedLogons / totalLogons, 2)
      | sort -privilegeRatio
      
  4728 4732 - Added to admin group
    --Use & Limitations
      Tracks when a user is added to any privileged group
      Specifically the Administrators group in the query below, additional target groups can be added
      
    --Query
      index=botsv3 sourcetype=WinEventLog:Security (EventCode=4728 OR EventCode=4732)
      | where Group_Name in ("Administrators")
      | table _time Account_Name Group_Name Security_ID ComputerName
      | sort -_time

Persistence
  4720 - user created
    --Use & Limitations
      Shows when a new user is created
      Many false positives are expected, and you should set up filters for creatorNames or other identifiable parameters
      Data set example, a user creates a fake svc account and shortly after adds it to the administrator group
      
    --Query
      index=botsv3 sourcetype=WinEventLog:Security EventCode=4720
      | eval creatorName = mvindex(Account_Name,0)
      | eval newUserName = mvindex(Account_Name,1)
      | table _time creatorName newUserName ComputerName
      | sort -_time
  
  4697 - Service Installed
    --Use & Limitations
      Shows when a service is created by an unusual user, like a non-service account
      Or if the service has an unusual name, like update/defender/svchost/windows
      Or if the service is from an unusual path, like users/programs/temp
      Adjustable parameters(users, path, and name) based on what is needed
      
    --Query
      index=botsv3 sourcetype=WinEventLog:Security EventCode=4697
      | eval suspiciousUser = if(Account_Name LIKE "%$", "NO", "YES")
      | eval suspiciousPath = if(
        match(Service_File_Name,"(?i)\\\\users\\\\|\\\\programdata\\\\|\\\\temp\\\\"),
        "YES", "NO"
      )
      | eval suspiciousName = if(
        match(Service_Name,"(?i)update|defender|svchost|windows"),
        "YES", "NO"
      )
      | where suspiciousUser="YES" OR suspiciousPath="YES" OR suspiciousName="YES"
      | table _time suspiciousUser Account_Name suspiciousName Service_Name suspiciousPath Service_File_Name ComputerName
      | sort -_time
