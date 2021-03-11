---
layout: post
title: New Kid on The Block, Babuk
categories: [Malware, Ransomware, Babuk]
excerpt: A brief history and overview of ransomware leading up to the newest addition to the ransomware family in early 2021, known as Babuk.  
---

(Estimated Reading Time: 8 minutes)

- [Ransomware History](#ransomware-history)
- [Babuk](#babuk)
- [References](#references)

## Ransomware History

Ransomware is a type of malicious software that attackers have engineered to convince or force the recipient of the ransomware to pay out large sums of money to the attacker. Successful ransomware has either scared the user into paying, often called Scareware, or has encrypted all the users files demanding payment to get their files back (unencrypted). Ransomware is not new and has been around for well over a decade with new variants continuing to emerge. A picture is worth a thousand words so the image below just shows a high level view of some well known ransomware families from the past decade. More recently in 2019 the Ryuk ransomware family had some large payouts. In 2020 there was a spike in ransomware largely driven by the global pandemic, a bit of a ransomware gold rush of sorts. The top 5 most active ransomware families in 2020 were Maze, Conti, Egregor, DoppelPaymer, and REvil. 2021 is kicking off with a new kid on the block, Babuk ransomware. 

![ransomwarehistory](/images/ransomwarehistory.jpg)

Ransomware-as-a-Service (RaaS) has become a thing in the past 1-2 years as organized crime groups have offered up leasing ransomware to affiliates for further network compromise. The profits are shared between the operators and affiliates. RaaS is changing the game a bit as affiliates without the knowledge to create malicious ransomware can simply buy/rent it just as easily as average consumers buy Cloud services. 

How big are these ransomware payouts you might ask? In 2019 the average payout was around $41,000 which jumped to an average payout of $233,817 in Q3 of 2020 [[Source](https://www.coveware.com/blog/q3-2020-ransomware-marketplace-report#:~:text=Average%20Ransomware%20Increases%20as%20Attackers%20Target%20Bigger%20Companies&text=The%20average%20ransom%20payment%20increased,to%20drag%20the%20averages%20up.)]. Most large payouts tend to be from corporate or service type organizations that have been targeted, like schools, hospitals, and utilities. The following table shows data traced through the bitcoin network showing the top 15 ransomware families by bitcoin payment. The graph and research is from [Ransomware in the Bitcoin Ecosystem](https://academic.oup.com/cybersecurity/article/5/1/tyz003/5488907)

![payouts](/images/payouts.jpg)

Now that we know a bit of the history of ransomware let's take a quick peak at the newest addition, Babuk. 

## Babuk

First seen in the wild on [VirusTotal on 4-Jan-2021](https://www.virustotal.com/gui/file/8203c2f00ecd3ae960cb3247a7d7bfb35e55c38939607c85dbdb5c92f0495fa9/details), the Babuk ransomware family is following similar tactics as Ryuk and Sodinokibi. McAfee has already completed a detailed analysis of one Babuk sample and authored a detailed [Technical Analysis of 
Babuk Ransomware](https://www.mcafee.com/enterprise/en-us/assets/reports/rp-babuk-ransomware.pdf) which is a good read. The file analyzed by McAfee has a SHA256 hash:

```8203c2f00ecd3ae960cb3247a7d7bfb35e55c38939607c85dbdb5c92f0495fa9```

I decided to look a bit more into this sample. Even though it has been analyzed by McAfee and others this sample is nice because the original variant is not packed and can be easily downloaded from [Tria.ge](https://tria.ge/210103-gpzkfkh3ej) or [VirusShare](https://virusshare.com/).

The ransomware letter from this sample can be seen in plain text by just looking at the Strings as seen below. I used to DetectItEasy in this case.  

![ransomletter](/images/ransomletter.jpg)

I ran the sample in a virtual machine and did in fact end up with encrypted personal files. Any files in the virtual machine that was user writable was encrypted (My Documents, My Downloads, etc.). Be careful as any network mapped file shares would also be encrypted! The OS files remained untouched and I was able to use Windows as normal. Any newly created files were fine as the Babuk ransomware doesn't persist or add any AutoRun registry keys or scheduled tasks that I could identify.

As called out in the McAfee report, I did see vssadmin being called to wipe out any shadow volumes that could be used for recovery.

![vssadmin](/images/vssadmin.jpg)

The table below is not an exhaustive list of samples but shows babuk over time as submitted to Trig.ge. Interestingly the 196k sample did introduce UPX packing, and the sample from 22-Feb that was 75k, changed the ransom note string from babuk to babyk ransomware. 

| Hash  | First Seen in Tria.ge  |  Size |
|---|---|---|
| [8203c2f00ecd3ae960cb3247a7d7bfb35e55c38939607c85dbdb5c92f0495fa9](https://tria.ge/210103-gpzkfkh3ej)  | 3-Jan-2021  |  30k | 
| [30fcff7add11ea6685a233c8ce1fc30abe67044630524a6eb363573a4a9f88b8](https://tria.ge/210108-8k176xp1zj)  | 8-Jan-2021  |  31k |
| [550771bbf8a3e5625d6ec76d70ed86f6e443f07ce80ff73e47f8249ddd72a8cf](https://tria.ge/210118-4myp12qpga)  | 18-Jan-2021  |  22k |
| [8140004ff3cf4923c928708505754497e48d26d822a95d63bd2ed54e14f19766](https://tria.ge/210118-4ydpsw46y2)  | 18-Jan-2021  |  196k |
| [1b9412ca5e9deb29aeaa37be05ae8d0a8a636c12fdff8c17032aa017f6075c02](https://tria.ge/210119-vy4d74cvsn)  | 19-Jan-2021  |  31k |
| [550771bbf8a3e5625d6ec76d70ed86f6e443f07ce80ff73e47f8249ddd72a8cf](https://tria.ge/210118-4myp12qpga)  | 22-Jan-2021  |  18k |
| [afcf265a1dcd9eab5aab270d48aa561e4ddeb71c05e32c857d3b809bb64c0430](https://tria.ge/210122-dcvdsabdme)  | 22-Jan-2021  |  38k |
| [3dda3ee9164d6815a18a2c23651a53c35d52e3a5ad375001ec824cf532c202e6](https://tria.ge/210125-bk7nj8slxx)  | 25-Jan-2021  |  30k |
| [58ccba4fb2b3ed8b5f92adddd6ee331a6afdedfc755145e0432a7cb324c28053](https://tria.ge/210128-b3gdrfb24s)  | 28-Jan-2021  |  29k |
| [391cfcd153881743556f76de7bbca5b19857f8b69a6f6f6dfde6fd9b06c17f5e](https://tria.ge/210222-mfkm1zpavj)  | 22-Feb-2021  |  75k |

[All Babuk samples on Tria.ge](https://tria.ge/s?q=family%3Ababuk)

[MalwareBazaar also has samples](https://bazaar.abuse.ch/browse/tag/Babuk/) matching the Babuk family.

The upx sample can be easily unpacked with the [upx packer](https://upx.github.io/) using the commend 'upx -d filename'. After unpacking the file is 1,077Kb and contains over 1200 strings, where the original only had 270 strings. In this unpacked sample idapro finds 846 functions where the original had 76. There seems to be more changes in this sample then just packing. The sample matches the Microsoft Visual C++ 9.0 compiler from Visual Studio 2008, which is also different then the original variant. 

I've created a high level pseudo execution flow of Babuk just to show at a very high level the simple flow of execution. I've omitted much of the Thread safety flow, and the specific encryption routines. At a glance this is what I think Babuk is doing after reviewing the Ghidra disassembly. 

```
if (not mutex)   //determine if Babuk has already  run on this box  
    call GetCommandLineA   //look for 'nolan' or 'lanfirst' command line args  
    call SetProcessShutDownParameters(0,0)  
    call OpenSCMangerA //get a list of services  
        foreach service  
            iterate services and terminate   specific hard coded services
    call Process32FirstW
        foreach (process)
            iterate running processes and terminate specific hard coded processes
    SHEmptyRecycleBinA   //clear recycling bin
    foreach (share)
        iterate over shares/files
    call GetSystemInfo //to get number of processors 
    call CreateThread 
    if (lanfirst command line)
        call WNetOpenEnumW   //enumerate network resources
        iterate over shares and files
        encrypt files
        CreateFileW //create ransomware note
    call GetLogicalDrives  //get local drives (ex C:)
    if (logicaldrives)
        iterate over files
        encrypt files
        CreateFileW //create ransomware note
    call ShellExecuteW to run vssadmin.exe delete shadows /all /quiet
else
  exit
```

The McAfee report indicates a version of Babuk may be available for *nix as well. Has anyone seen a sample from a *nix machine? If so let me know in a comment below. 

## References

* Link to [McAfee Technical Analysis of Babuk](https://www.mcafee.com/enterprise/en-us/assets/reports/rp-babuk-ransomware.pdf)
* Link to [NHS Babuk Guidance and IOC's](https://digital.nhs.uk/cyber-alerts/2021/cc-3715)
* Link to ReversingLabs YARA Rule [Win32.Ransomware.Babuk.yara](https://github.com/reversinglabs/reversinglabs-yara-rules/blob/develop/yara/ransomware/Win32.Ransomware.Babuk.yara)
* Link to [Babuk Sample on Tria.ge](https://tria.ge/210103-gpzkfkh3ej)
* Link to [Babuk Sample on VT](https://www.virustotal.com/gui/file/8203c2f00ecd3ae960cb3247a7d7bfb35e55c38939607c85dbdb5c92f0495fa9/details)
* Link to [Malwarebytes Types of Ransomware](https://www.malwarebytes.com/ransomware/#typesofransomware)