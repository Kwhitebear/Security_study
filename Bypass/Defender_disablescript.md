# Defender_disablescript

Before entering, disable Defender Script is currently being detected by Windows Defender, so an appropriate bypass is required.<br>

This article will write about closing Windows Defender using the registry<br>

Note that everything must be done with elevated privileges.<br>

# Registry

```
reg add "HKLM\Software\Policies\Microsoft\Windows Defender" /v "DisableAntiSpyware" /t REG_DWORD /d "1" /f
reg add "HKLM\Software\Policies\Microsoft\Windows Defender" /v "DisableAntiVirus" /t REG_DWORD /d "1" /f
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/89289e3f-05a3-4ca6-939c-7a03fd6c054f)

DisableAntiSpyware & DisableAntiVirus Value Set 1.<br>
A value of 1 means disable spyware and virus detection.<br>

```
reg add "HKLM\Software\Policies\Microsoft\Windows Defender\MpEngine" /v "MpEnablePus" /t REG_DWORD /d "0" /f
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/26183974-3efc-4f00-8ad2-395127be2394)

Create Folder "MpEngine", MpEnablePus Value set 0.<br>
A value of 0 means Disable Windows Defender's unauthorized file upload feature.<br>

```
reg add "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" /v "DisableBehaviorMonitoring" /t REG_DWORD /d "1" /f
reg add "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" /v "DisableIOAVProtection" /t REG_DWORD /d "1" /f
reg add "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" /v "DisableOnAccessProtection" /t REG_DWORD /d "1" /f
reg add "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" /v "DisableRealtimeMonitoring" /t REG_DWORD /d "1" /f
reg add "HKLM\Software\Policies\Microsoft\Windows Defender\Real-Time Protection" /v "DisableScanOnRealtimeEnable" /t REG_DWORD /d "1" /f
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/391f22ce-197b-429d-8fce-5936c4b407ba)

Create Folder "Real-Time Protection", everything Value set 1.<br>
- DisableBehaviorMonitoring : Disable Windows Defender's Behavior Monitoring feature. Behavior monitoring is the role of detection and blocking the behavior of malicious code.
- DisableIOAVProtection : Disable Windows Defender's input and output(I/O) protection feature. I/O protection detects and blocks malicious file read and write attempts.
- DisableOnAccessProtection : Disable the block-on-access feature of Windows Defender. Block on access plays a role in detection and blocking attempts to access malicious files.
- DisableRealtimeMonitoring : Disable Windows Defender's real-time monitoring feature. Real-time monitoring monitors processes and files running on the system in real time to block malicious behavior.
- DisableScanOnRealtimeEnable : Disable scanning when Windows Defender's real-time activation occurs. Scan on real-time activation serves to scan the system for amlware when Windows Defender is activated.

<br>

```
reg add "HKLM\Software\Policies\Microsoft\Windows Defender\Reporting" /v "DisableEnhancedNotifications" /t REG_DWORD /d "1" /f
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/854edcb3-bbfd-4289-bf57-6c9bdd2003e9)

Create Folder "Reporting", DisableEnhancedNotifications Value set 1.<br>

One of the settings in Windows Defender, the ability to control enhanced notifications.<br>
A value of 1 means "disable" the feature. So, this command disables enhanced notifications in Windows Defender.<br>

```
reg add "HKLM\Software\Policies\Microsoft\Windows Defender\SpyNet" /v "DisableBlockAtFirstSeen" /t REG_DWORD /d "1" /f
reg add "HKLM\Software\Policies\Microsoft\Windows Defender\SpyNet" /v "SpynetReporting" /t REG_DWORD /d "0" /f
reg add "HKLM\Software\Policies\Microsoft\Windows Defender\SpyNet" /v "SubmitSamplesConsent" /t REG_DWORD /d "0" /f
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/d74d7525-c940-43f9-a2ef-5d7cdd9a1a4d)

Create Folder "SpyNet"<br>

<strong>SpyNet
 - SpyNet is a cloud-based security service built as part of Windows Defender. So, SpyNet is a service that works in the background and is one of several componets that run alongside Windows Defender.<br>
 - SpyNet is used to collect and analyze information about files, behaviors, URLs, etc. related to malicious software to provide security updates and rapid response.
</strong>

<br>

<strong>Cloude Service
- Cloude services are computing resources and technology services delivered over the Internet. Traditionally, users run software and store data on their own computers or local servers, but with cloude services, these tasks can be processed on external servers and resources over the Internet.
</strong>

<br>

- DisableBlockAtFirstSeen : Disables SpyNet's ability to block the first file it finds. A value of 1 means "disable" the function.
- SpynetReporting : Controls SpyNet's ability to submit reports. A value of 0 means "disable": report submission.
- SubmitSamplesConsent : Control whether to consent to submit sample files. A value of 0 means "disable": consent to submit sample files.

<br>

```
reg add "HKLM\System\CurrentControlSet\Control\WMI\Autologger\DefenderApiLogger" /v "Start" /t REG_DWORD /d "0" /f
reg add "HKLM\System\CurrentControlSet\Control\WMI\Autologger\DefenderAuditLogger" /v "Start" /t REG_DWORD /d "0" /f
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/05877b5c-1227-447f-a7b6-c301d78c5c8b)

disables Windows Defender's API logging and audit logging.<br>

<strong>
API logging refers to recording information about application programming interface(API) calls executed aby an application or system. APIs are functions or methods used by programs to interact with the operating system, libraries, services, etc., enabling communication and data exchange with other software components.
</strong>

<br>

- DefenderApiLogger : This location is the registry path where settings for Windows Defender's API logging are stored. Set the "Start" value to 0 to "disable" that logging. API logging is used to record log information about the operation of Windows Defender.
- DefenderAuditLogger : This location is the registry path where the settings for Windows Defender's audit logging are stored. Set the "Start" value to 0 to "disable" that logging. Audit logging records log information about Windows Defender's actions and events.

<br>






