---
layout: post
title: Cry Me A River oh DearCry
categories: [Malware, Ransomware, DearCry]
excerpt: Overview of new DearCry ransomware and some artifacts from reverse engineering a couple DearCry samples.  
---

(Estimated Reading Time: 6 minutes)

- [DearCry](#dearcry)
- [References](#references)

## DearCry

Hot off the press, DearCry has emerged as a new ransomware threat over this past weekend. Threat actors are using the recent Exchange vulnerabilities, [ProxyLogon](https://www.microsoft.com/security/blog/2021/03/02/hafnium-targeting-exchange-servers/#scan-log), as the vector to initiate DearCry ransomware. DearCry is also known as doejocrypt by many AV vendors.

There are 7 samples of DearCry available on [Bazaar](https://bazaar.abuse.ch/browse/tag/DearCry/). I looked at these two samples in Ghidra.

```
e044d9f2d0f1260c3f4a543a1e67f33fcac265be114a1b135fd575b860d2b8c6

feb3e6d30ba573ba23f3bd1291ca173b7879706d1fe039c34d53a4fdcdf33ede
```

For starters, the samples have just over 4,500 text strings that are easily readable. The ransomware does not appear to be packed. Most the 4,500 strings appear to be a result of OpenSSL being compiled into the binary I'm thinking. There are only about a dozen or so strings that seem interesting as seen below.

![dearcrystr1](/images/dearcrystr1.jpg)
![dearcrystr2](/images/dearcrystr2.jpg)

The ransom note string text and the file it outputs to, readme.txt can be clearly seen. The .crypt string is used as part of the file extension for files that have been encrypted. The msupate string is the service name created for the NtService the performs the background encryption of files. The list of file types to be encrypted can also be clearly seen. 

```
.TIF .TIFF .PDF .XLS .XLSX .XLTM .PS .PPS .PPT .PPTX .DOC .DOCX .LOG 
.MSG .RTF .TEX .TXT .CAD .WPS .EML .INI .CSS .HTM .HTML  .XHTML .JS 
.JSP .PHP .KEYCHAIN .PEM .SQL .APK .APP .BAT .CGI .ASPX .CER .CFM .C 
.CPP .GO .CONFIG.CSV .DAT .ISO .PST .PGD  .7Z .RAR .ZIP .ZIPX .TAR 
.PDB .BIN .DB .MDB .MDF .BAK .LOG .EDB .STM .DBF .ORA
```

The strings for WINDIR, APPDATA and PROGRAMFILES are likely used to exclude those system directories from encryption. When looking for references in Ghidra I only saw WINDIR being used in a call to the _getenv function which on most computers will return C:\Windows. I would have expected a similar call for appdata and programfiles but didn't see that in the sample I looked at. Below you can see the call to _getenv("WINDIR").

![dearcry](/images/dearcry.jpg)

From other post's I've read folks are calling DearCry a "copy" ransomware because it creates encrypted copies of the attacked files. It encrypts the copied files and then deletes the original version of the file. In some cases it may be possible to recover some of the deleted files with forensics tools, however it is also very likely that much of that data will be overwritten or just unrecoverable.

The very first things this ransomware does is *try* to setup a service called msupdate by passing the service name and a pointer to the services main function to the call StartServiceCrtlDispatcherA

![dearcry](/images/dearcryservice.jpg)

If the StartServiceCrtlDispatcherA fails the next function (00401D10) is the same function used inside the NtService so either way the same code runs just depends if it runs in a service or not. 

If the service gets created you will be able to easily spot this in the Windows Event Log or tools like Splunk by looking for the new windows service event 7045. In that event you'll see the service name msupdate in this case. That should be a red flag for the Security Operation Centre (SOC) to investigate. 

| Event | Event ID |  Level | Event Log |
|---|---|---|---|
| New Windows Service | 7045 | Information | System |
||||

If the service is created, and a service handler is registered the main service function is called and we start out at function (00401D10) which starts out by getting the WINDIR environment variable, and calling sprintf to build the ransom note for readme.txt. It then loads the RSA key and calls GetLocalDrives to enumerate local drives (like C:\, D:\, etc). Some other functions are then called that check the file extension against the list above. You can see a call graph below from what I called 00401D10_Cry_Main (0x00401D10).

![crymain](/images/crymain.jpg)

There are 2,509 functions in the sample I'm looking at. I suspect many of those are OpenSSL functions and calls to imported Windows cryptographic functions. 

At the end of the service main function the ransomware will call OpenServiceA on itself and then call DeleteService to delete itself. 

Stay safe out there, and get those Exchange Servers Patched ASAP....

## References

* Link to [Microsoft Exchange ProxyLogon](https://www.microsoft.com/security/blog/2021/03/02/hafnium-targeting-exchange-servers/#scan-log)
* Link to [DearCry Samples on Bazaar](https://bazaar.abuse.ch/browse/tag/DearCry/)
* Link to [ReversingLabs DearCry YARA Rule](https://github.com/reversinglabs/reversinglabs-yara-rules/blob/develop/yara/ransomware/Win32.Ransomware.DearCry.yara)