This Defensive CTF is focused on Malware Analysis of a PowerShell script

Tools used:
  Notepad++
  CyberChef

Foundation:
  This resource is provided and will be useful for some questions: https://www.trendmicro.com/en_us/research/24/k/earth-estries.html
  There are 2 lines in the script. I cleaned them up and saved the first line as main.txt, and the second line is as decryptor.txt

1. What is the name of the malware family associated with the provided file?
  In the article, it mentions the malware family a few times
  "Demodex"

2. What .NET cryptographic class is used to perform decryption in the script?
  On the second line of the cleaned-up decryptor script, there is a base64 encoded string that decodes to
  "System.Security.Cryptography.AesManaged"

3. The key to decrypt the script must be entered on the command line when running the ps1. What variable holds the key?
  In the cleaned-up decryptor script, the first line assigns $args[0] to $k.
  I confirmed this as well since line 4 declares the decryptor key as $k
  "$k"

4. What is the key required to decrypt the base64 encoded data?
  You can find this in the article
  "password@123"

--Note--
  Decrypting the main script is simple since all the parameters are in plaintext; make sure you pad the key. Here is my CyberChef recipe.
  https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)AES_Decrypt(%7B'option':'UTF8','string':'password@12300000000000000000000'%7D,%7B'option':'UTF8','string':'0000000000000000'%7D,'CBC','Raw','Raw',%7B'option':'Hex','string':''%7D,%7B'option':'Hex','string':''%7D)&oenc=65001&ieol=CRLF&oeol=NEL
  
5. After decrypting the initial payload, a new PowerShell script assigns a value to the variable $cregvalue. What is that value?
  In main.txt, you will see $cregvalue = 
  "midihelp"

6. The variable $cregdata is associated with binary registry data stored as a base64 blob. What is the SHA-256 hash of the binary data?
  This value is stored right below the $cregvalue. Using CyberChef, create a recipe that decodes "from base64", then "SHA2" with a size of 256, and 64 rounds
  "3b1c251b0b37b57b755d4545a7dbbe4c29c15baeca4fc2841f82bc80ea877b66"

