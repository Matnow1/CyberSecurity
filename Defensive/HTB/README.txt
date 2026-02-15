Tools Used:
  Autopsy
  MFTECmd
  MFT Explorer
  Event Viewer
  unixtimestamp.com

Foundation:
  For simplicity, open the C drive in Autopsy as a logical file, this will allow easy reading of most files that would normally require a special program

1. What is the Computer name of the machine?
  In autopsy under Data Artifcats/Operating System Information, you will see logicalfileSet# and the name
  "InvisibleChains"

2. What is the first Wi-Fi SSID(Decoded) they connected to on May 30th 2025?
  Under C\Windows\System32\winevt\logs, use Event Viewer to open Microsoft-Windows-WLAN-AutoConfig%4Operational.evtx
  Filter by eventID 8001 and look for the first entry on 30-05-2025, you will see a connection to
  "ArboretumCoffee"

3. When did the system obtain a lease for the network?
  Under C\Windows\System32\Config, Select SYSTEM and go to ControlSet001\Services\Tcpip\Parameters\Interfaces
  The right folder is {18c11dbd-93ab-4ca9-a804-4f4475da25b8}, convert its LeaseObtainedTime from Unix, and it is
  "2025-05-30 18:22:48"

4. What IP address did the device receive when connecting to the café?
  In the same folder as above, look at the DhcpIPAddress it is
  "172.16.100.16"

5. What was the BSSID (MAC address) of the access point they connected to at the café?
	In the same file as the IP address, go down to the DhcpGatewayHardware
	The last 12 characters are the MAC address 
  "E4-D1-24-96-A5-D1"
	
6. It looks like they started some sort of manifesto at the café, what is the name of the file they started to write?
	Go to  C\Users\Ernes\NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
	You can see one text file, and it is a little after the time they got the lease for the network
	"The Chains Not Seen.txt"
	
7. What is the last sentence of the manifesto?
	Use MFTECmd on the MFT file, then look for the filename. Once you find it, remember the location
	$MFT\Users\Ernes\OneDrive\Desktop
	In MFT Explorer, go to the location, click on the file, and in the data, you can see the contents in various formats, the last line is 
  "Freedom is a perspective away."
	
8. They started their research by watching a YouTube video of a speech, what is the name of the speech?
	Go to C\Users\Ernes\AppData\Local\Microsoft\Edge\User Data\Default, open history with DBBrowser
	Go to the URLs table, filter title by YouTube, and you can see 
  "The Ballot or the Bullet"
	
9. They continued their research by looking up a book on Wikipedia, what was the title of the book?
	Change the title filter to Wikipedia, and you will see 
  "The Iron Heel"
	
10. What was the last thing they downloaded before leaving the café?
	Switch tables to downloads, and you can see two entries; the last one is 
  "BraveBrowserSetup.exe"
	
11. When investigating changes to network profiles on a Windows system, which event log would you examine to find entries related to these profile-specific events, using event IDs such as 10000 or 10001?
	Event logs are specifically located in C\Windows\System32winevt\logs, the specific one for network profiles is
  "Microsoft-Windows-NetworkProfile/Operational"
	
12. sing the logs from the previous answer, when did they disconnect and leave the first café?
	Open the previous event logs in Event Viewer, then filter by disconnections (EventID 10001)
	Sort by newest first, and you will see the second cafe twice, then the third row is the last disconnection from the first cafe
	(Make sure you go to the details tab and use the systemTime)
  "2025-05-30 18:55:45"
	
13. Using the same logs, when did the user arrive at the second café?
	Change the filter to connections (EventID 10000)
	Sort by newest first, and I went to the rough time of the first cafe disconnection
	Then find the first result that has an identified network
	(Make sure you go to the details tab and use the systemTime)
  "2025-05-30 19:05:26"
	
14. What is the SSID(decoded) of the second Wi-Fi they connected to on May 30th 2025?
	I have already seen it a bunch of times, but you can use the previous logs and just look at the name
	"Happy Trails Guest"
	
15. What IP address did the device receive when connecting to the second café?
	Open the TcpIP file again(exact path under q3)
	Go to the same interface as before, and then look for the most recent modification time after the connection time, then look at the DhcpIPAddress 
  "192.168.10.100"
	
16. When did the system obtain a lease for the second network?
	Put the LeaseObtainedTime unix timestamp(the value inside parentheses) into a website like https://www.unixtimestamp.com/
	"2025-05-30 19:06:02"
	
17. What was the BSSID (MAC address) of the access point they connected to at the second café?
	Again, exactly the same as the first cafe, in the same file, look at the DhcpGatewayHardware and copy the last 12 characters 
  "4C-BA-7D-E1-8C-30"
	
18. What was the first thing the user downloaded at the second café?
	We know the user went to the first cafe to download Brave Browser
	Since there are no other downloads on Edge, we will go to the Brave history instead
	C\Users\Ernes\AppData\Local\BraveSoftware\Brave-Browser\User Data\Default
	Open History with DBBrowser, Go to downloads table, the first download is 
  "VirtualBox-7.1.8-168469-Win.exe"
	
19. What online forum/social media site did they visit?
	Go to the URLs table, and the only forum site is 
  "reddit"
	
20. What was their username on the site?
	Exit the database, then go to Login Data, then the logins table.
	The only result is for Reddit, and their username is 
  "FinanciallyFree3636"
	
21. What was the name of the VM they created?
	$MFT\Users\Ernes\VirtualBox VMs, the folder is the name 
  "LastHope"
	
22. What street was the first café located on?
	Look up the cafe name on Google. The street is 
  "W Prospect Rd"
	
23. Investigators may want to follow up on the Wi-Fi credentials used at the first café the suspect visited. Which file stores the authentication details (including the encrypted password) for the first network?
	You can find the authentication data under C\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces\{18C11DBD-93AB-4CA9-A804-4F4475DA25B8} the specific connection is 
  "{BAC95378-DC6B-4464-918E-4E005F747786}.xml"
	
24. What authentication method was used to connect to the first café's Wi-Fi?
	Open the file from the previous question, and you will see 
  "WPA2PSK"
