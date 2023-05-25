# MsMpEng.exe

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/5ceef7cd-6613-4c05-9c73-7a755922e4c7)

One of the main components of Windows Defender, the continuous monitoring process "MsMpEng.exe"

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/e6721f1e-a01b-403c-8130-1b5af4cfc4ca)

mpengine.dll 
- Microsoft Malware Protection Engine(MP Engine) : It is a Key component included in Microsoft's security products such as <strong>Windows Defender</strong> and <strong>Microsoft Security Essentials</strong>.
This engine is used to provide malware detection and removal, scanning capabilities, real-time protection and other security features.<br><br>  


<strong>CVE-2021-1647</strong><br>
https://www.bleepingcomputer.com/news/security/microsoft-patches-defender-antivirus-zero-day-exploited-in-the-wild/<br>

This News Remote Code Execution Vulnerability in mpengine.dll, a Microsoft Malware Protection component.

Upx/NSPacker/Shrinker/PECompact2/Crypter1337/Aspack/PKLite/ASprotect, etc. This vulnerability occurs in the <strong>Asprotect</strong> simulation decompression 
process(CAsprotectDLLAndVersion::RetrieveVersionInfoAndCreateObjects function).<br>


![image](https://github.com/Kwhitebear/Security_study/assets/99308681/100c3a9c-3d90-41ac-9c64-961a80ea9d59)

v2+28 represents the start of a section array containing 4 elements, each section element is 8 bytes, the first 4 bytes are used to describe the virtual address of the section 
and the last 4 bytes are used. To illustrate the size of the section, the picture above illustrates the process of: It traverses the section array containing 4 items and finally gets
the "sectionval" and "sectionszie". Apply a piece of memory of size "sectionva+sectionsize" to store the decompressed section content. but it has an error in the calculation of sectionva and sectionsize.
If the values of the 4 elements of the section array are [0,0],[0,0],[0x2000,0],[0x2000,0x3000] according to the logic of the code above, if sectionva = 0x2000, sectionszie =0, final request THe memory
size is 0x2000 + 0 = 0x2000, so when unpacking the alst section, the size is 0x3000, so have a heap overflow.<br>

<strong>more imformation : https://www.anquanke.com/post/id/231625</strong><br><br>


# Notice

1. In the Windows account, "TrustedInstaller" is the default system account of the Windows operating system, and this account is used by system services such as Windows Defender.<br>

2. Token<br>
 -Tokens within the Microsoft Windows operating system are secure elements that provide identification to processes and threads when attempting to perform operations on securable objects(filoes, records, services..) in the System.
In other words, it's like showing your ID when you try to enter.<br>

