This Defensive CTF focuses on analyzing a smishing attack that lead to a malicious apk being downloaded

Tools Used:
	Autopsy
	https://www.epochconverter.com/webkit
	apktools
	https://www.unixtimestamp.com
	jadx-gui
	Wireshark
	CyberChef

Foundation:
  	For simplicity, open the C drive in Autopsy as a disk image. This will allow easy reading of most files that would normally require a special program
    Set the timezone to UTC+0

1. Provide the UTC timestamp of the phishing SMS. 
	Under Data Artifacts -> messages, I saw 4 greetings, then a 5th message from a bank about a transaction, and to confirm it, you need to update the app from a specific URL.
	This is a very suspicious message. The time of the message is under the Date/Time column
	"2025-02-01 16:20:32"

2. Provide the UTC timestamp marking the start of the malicious application download.
	The download is a link, not in an app store, so I checked Chrome's download history in Autopsy by going to
	android-9.0-r2/data/data/com.android.chrome/app_chrome/Default/History, then I opened the application tab and the downloads table.
	I got a start time of 13382903003690820. This is in webkit time, so I used a converter(https://www.epochconverter.com/webkit)
	"2025-02-01 17:03:23"

3. Provide the package name of the malicious application.
	From the Chrome download history, I kept a note of the download destination(android-x/data/media/0/Download/S1rBank.apk)
	I found the download and extracted it from Autopsy. I opened a CMD prompt in the folder where it was extracted to and ran "apktools d .\S1rBank.apk"
	Then go to the extracted folder and open the AndroidManifest.xml, and you will see package = 
	"com.s1rx58.s1rbank"

4. Provide the number of runtime permissions granted to the malicious application.
	In Autopsy, go to android-x/data/system/users/0/runtime-permissions.xml, and open it in an external window
	Then look for the package name identified in Q3 and count the permissions
	"4"

5. Provide the last access timestamp for the read sms permission used by the malicious application.
	In Autopsy, open the android-x/data/system/appops.xml in external view
	Ctrl+f the package name in Q3, You will have to know the operation integer to string correlation which varies by android version.
	In this version, 14 is equal to READ_SMS, so convert the Unix timestamp to human human-readable format and get 
	"2025-02-01 17:07:18"

6. Provide the URL used by the malware for data exfiltration.
	Using jadx-gui, open the .apk and then go to Source Code/com/s1rx58.s1rbank
	You get 4 classes; R can be ignored. MyBackgroundService handles the C2 creation
	"http://s1rbank.net:80/api/data"

7. The malicious application checks if the server is live before sending data. Provide the HTTP method used for this check.
	The first function in MyBackgroundService, "a", creates a simple connection to the C2 server and returns true if it is successful
	The second line says the method it uses,
	"HEAD"

8. If the primary server is unavailable, the malicious application redirects data exfiltration to an alternate URL. Identify and provide the alternate URL.
	Checking the usage shows that IO.c.doInBackground uses the connection check function "a".
	If the connection check passes, it uses the other function in MyBackgroundService ("b") to send a message.
	If the connection fails, it goes to B.h.IO, which has a hardcoded Discord webhook as a backup
	"https://discord.com/api/webhooks/1334648260610097303/-Lkxr0eZRO_fb_SaumBbBMZyANM3lyeCkR-E1NXXRASPbtRdNksQSzx4pY1ZGQkFR2H8"

9. The malicious application encrypts data before sending it to the server. Provide the encryption key used.
	From IO.c.doInBackground the message is the variable bArrEncode which is base64 encoded message.
	The message is in plaintext above in the variable sb, but it is also encrypted in B.h.C
	And in B.h.C the key is in plaintext
	"0x_S1r_x58!@#53cuReK371337!$%^&*"

10. Credit card information was stolen. What was the second line in the exfiltrated payment information?
	In the pcap we can find a few calls to the c2 server but the data is encrypted and encoded.
	I set up a cyberChef recipe that first uses "From Base64" and then AES decrypt with EDB and the key from Q9 as UTF8. (https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)AES_Decrypt(%7B'option':'UTF8','string':'0x_S1r_x58!@%2353cuReK371337!$%25%5E%26*'%7D,%7B'option':'Hex','string':''%7D,'ECB','Raw','Raw',%7B'option':'Hex','string':''%7D,%7B'option':'Hex','string':''%7D)&input=L05WRVF3ZVpxRTNjOEJEeFR2MHZ3Ynd2OEF4MVJ5MEs2K21VVTdkeVVqaWNPdTdQRW5yUEhJRW1VZS83Y2NmS21MS21TMlF3SVFZTjhibTV4aDY0c0l6eWt5UUhXc05QS0Rud3lRZkNObEtuczZ2UGxPbUpvYzh2MDAzbmJxWkwNCg&ieol=CRLF)
	Back in Wireshark using the filter ' http.request.full_uri == "http://s1rbank.net/api/data" ' we can see all the c2 requests.
	As expected, for every message, there is a connection check with the HEAD method
	The first post request is about card details, the second post is every message recorded, and the third one was blank.
	Since the question is about credit card information, we are interested in the first post message, and the second line is 
	"Card Number: 5453004085527987"
