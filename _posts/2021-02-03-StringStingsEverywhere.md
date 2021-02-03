---
layout: post
title: Strings Strings Everywhere
categories: [Malware, Strings, Tools]
---
(Estimated Reading Time: 11 minutes)

- [Why Look at Strings](#why-look-at-strings)
- [Example of Looking at Strings](#example-of-looking-at-strings)
- [References](#references)

## Why Look at Strings

Although it is not common for malware samples to contain clear text strings, it does happen, and it is an easy place to start analysis. Strings can be embedded in malware samples and provide indications to what the malware will do when executed. Strings can be looked at statically using tools without requiring the malware to be run. 

I decided to take a benign sample from an online malware course, [Lab01-01.dll](https://www.virustotal.com/gui/file/f50e42c8dfaab649bde0398867e930b86c2a599e8db83b8260393082268f2dba/details), and run it through different string tools to compare the tools. The string tools I used are referenced below. 

## Example of Looking at Strings

To start out my analysis I surveyed several tools for dumping string information which I have summarized in the table below. I then ran each tool against my sample, Lab01-01.dll, and reviewed the output which I have summarized in the second table below. 

| Tool | Type | Pro/Cons |
|---|---|---|
| strings.py | Python | Lots of great command line options, needs python but that is easy to install. Updated frequently with new features. |
| Strings2.exe | Executable | I could not get this tool to run with a long input filename path. When I copied the sample into the same folder as strings2.exe it worked. Standalone tool which is nice but not updated since 2017 it looks like. |
| BinText.exe | Executable | GUI tool that is easy to use, but strangely listed strings twice for some reason, more details on this below. Also looks like it has not been updated in a long time.  |
| IdaPro | Executable | GUI tool or IDAPython. Really over kill just for looking at strings but will do the job. |
| PEStudio | Executable | GUI tool with nice tips that help indicate why certain strings might indicate malicious behavior. Looks to be well supported and updated. Has a Pro version as well. |
| Hybrid-Analysis | Website | Easy to use web site but requires sample to be uploaded to the site. Will give you way more then just strings and may be overkill if only looking for a dump of strings  |
| FLOSS | Python | Looks like it would be good based on this [blog](https://www.fireeye.com/blog/threat-research/2016/06/automatically-extracting-obfuscated-strings.html), but I could not get it to work. Setup on Windows 10 x64 failed with an error of no Module named utils. |

Using the above tools with their default options against Lab01-01.dll found the following number of strings.

| Tool | Strings Found | Notes |
|---|---|---|
| strings.py | 37 | Was fast and easy to use
| strings2.exe | 37 | Was a bit tricker to run, but got the same result as strings.py |
| BinText.exe | 60 | This was a strange case. I realized it was listing strings twice (doubling) so it really found 30 strings. See screen shot below showing the doubling. |
| IdaPro | 8 | The free version of Ida listed 8 strings. |
| PEStudio | 36 | Easy to use GUI with nice hints column that tries to tell you a little context about the string, like URL or file. See image below |
| HybridAnalysis | 126 | This site found [126 strings in this sample](https://www.hybrid-analysis.com/sample/f50e42c8dfaab649bde0398867e930b86c2a599e8db83b8260393082268f2dba/5b1b09697ca3e1066c2f22e4). I think it used both static and dynamic analysis so it found additional strings at run time. |

BinText is shown here and finds strings twice for some reason. 
![BinText](/images/bintext.PNG)

PE Studio showing some hints about the context of a string.
![PEStudio](/images/pestudio.PNG)

Hybrid-Analysis showing a lot more strings than the static tools.
![hybrid_analysis](/images/hybrid_analysis.PNG)

In closing, all the tools found the strings 'hello' and 'sleep' which look like they could be commands. There is also a call to CreateProcessA and an IP address which could be a C&C server. If your looking for a good command line tool my favorite was strings.py by Didier Stevens, for a GUI tool I liked PEStudio. 

## References
* Link to [strings.py by Didier Stevens](https://blog.didierstevens.com/2021/01/24/update-strings-py-version-0-0-7/) 
* Link to [Strings2.exe by Geoff McDonald](http://split-code.com/strings2.html) 
* Link to [BinText by McAfee, I think](http://b2b-download.mcafee.com/products/tools/foundstone/bintext303.zip) 
* Link to [PEStudio by Winitor](https://www.winitor.com/) 
* Link to [Hybrid-Analysis](https://www.hybrid-analysis.com/) 
* Link to [FLOSS by Fireeye](https://github.com/fireeye/flare-floss) - Note: I could not get this tool to work on my Windows 10 x64 VM. If anyone got it working please let me know how at [@c00011011](https://twitter.com/C00011011)