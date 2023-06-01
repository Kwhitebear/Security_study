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

```
schtasks /Change /TN "Microsoft\Windows\ExploitGuard\ExploitGuard MDM policy Refresh" /Disable
schtasks /Change /TN "Microsoft\Windows\Windows Defender\Windows Defender Cache Maintenance" /Disable
schtasks /Change /TN "Microsoft\Windows\Windows Defender\Windows Defender Cleanup" /Disable
schtasks /Change /TN "Microsoft\Windows\Windows Defender\Windows Defender Scheduled Scan" /Disable
schtasks /Change /TN "Microsoft\Windows\Windows Defender\Windows Defender Verification" /Disable
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/2900e232-f3bb-410e-97c3-9d4481a54bcf)
<br>
![1685592873135](https://github.com/Kwhitebear/Security_study/assets/99308681/618a15ff-56df-4623-b306-cd0254d35593)

The picture above is a normal task scheduler.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/f2d272e7-56c8-4704-8dd0-52f4dbefa06c)
<br>
![image](https://github.com/Kwhitebear/Security_study/assets/99308681/e77d9884-dd7f-4dab-9a3f-ddddc5c870de)

- ExploitGuard : ExploitGuard is a feature of Windows Defender that is used to protect the system from malicious software. This task serves to refresh and apply Mobile Device Management(MDM) policies.
- Windows Defender Cache Maintenance : This task is responsible for maintaining and cleaning Windows Defender's cache.
- Windows Defender Cleanup : This task cleans up Windows Defender's temporary files and unnecessary items.
- Windows Defender Scheduled Scan : This task is to regularly scan your system to detect and remove malicious software.
- Windows Defender Verification : This task checks the validity of Windows Defender's library and definition files.

<br>

For the library files and definition files of Windows Defender, see below.<br><br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/f757ef6d-dfc8-413f-923e-7dfc226d3e23)

```C:\ProgramData\Microsoft\Windows Defender\Definition Updates```

<br>


<strong>The mpengine_etw.dll file supports Windows Defender's performance monitoring, logging, and analytics capabilities, allowing security operations teams to monitor and respond to the security posture of systems.</strong>

- NisBackup folder : This folder is where you store backups of Windows Defender's definition files. Definitions files are data used to detect malicious software and threats. The NisBackup folder keeps older versions of definition files so that they can be restored from backups if needed.
- StableEngineEtwLocation folder : This folder is for storing files related to Windows Defender engine logging. Engine logging records log information about Windows Defender's inner workings and operations. The StableEngineEtwLocation folder stores files used for engine logging.
- The mpengine_etw.dll file in the StableEngineEtwLocation folder : The mpengine_etw.dll file is a DLL file used for engine logging in Windows Defender. This DLL file is responsible for tracking events for Windows Defender's engine activity and operation. Engine logging helps you record and analyze Windows Defender's operational status and detection activity.

<h3> More Information</h3>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/4112f440-581e-457e-861d-80ebf55615a4)

```C:\Windows\System32\Tasks\Microsoft\Windows\Windows Defender```

<br>

Windows Defender Cleanup File -> "<Command>C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.23050.3-0\MpCmdRun.exe</Command>"

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/088f649a-92a8-4796-89aa-f85a4e8505bd)


<br><br>

Windows Defender Scheduled Scan -> "<Command>C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.23050.3-0\MpCmdRun.exe</Command>"

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/15dcccae-faa8-4b62-8a3c-142ba72a7e95)

<br><br>

```
reg delete "HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\StartupApproved\Run" /v "Windows Defender" /f
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v "Windows Defender" /f
reg delete "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v "WindowsDefender" /f
```

The Run folder serves as a persistence mechanism to automatically run every bootup. Delete the value in the registry to prevent Windows Defender from running at boot time.

<br><br>

```
reg delete "HKCR\*\shellex\ContextMenuHandlers\EPP" /f
reg delete "HKCR\Directory\shellex\ContextMenuHandlers\EPP" /f
reg delete "HKCR\Drive\shellex\ContextMenuHandlers\EPP" /f
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/36ba46a2-1bc9-4ca3-85a2-36d13ee089a9)

The picture above is a normal task scheduler.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/e1a4f307-174d-4e3d-be53-067fd3939641)

- HKCR*\shellex\ContextMenuHandlers\EPP : Represents a context menu handler for any file type. A context menu handler defines the options displayed in the right-click menu for that file type. "EPP" refers to Windows Defender's Context Menu Handler.
- HKCR\Directory\shellex\ContextMenuHandlers\EPP : This path represents the context menu handler for the folder(Directory). Define the behavior for the menu that appears when you right-click on a folder
- HKCR\Drive\shellex\ContextMenuHandlers\EPP : This path represents the context menu handler for Drive. Define the behavior for the menu that appears when you right-click on a drive

 <strong>EPP(Context Menu Handler for Windows Defender) </strong><br>
 The EPP value (Example:{09A201-01930a90-....-.....}) unique identifier pointing to the registered program for that context menu handler.<br>
This identifider is associated with an entry registered in the registry and used to launch the corresponding context menu handler.<br>

Typically, this value represents the globally unique identifier (GUID) or class identifier(CLSID) of the registered program.<br>
GUIDs are used as unique identifiers, and each entry points to a specific program or component.<br>

Therefore, an EPP value such as "Example" is a value for identifying a registered program to which the corresponding context menu handler is connected.<br>

<br>

```
reg add "HKLM\System\CurrentControlSet\Services\WdBoot" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\System\CurrentControlSet\Services\WdFilter" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\System\CurrentControlSet\Services\WdNisDrv" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\System\CurrentControlSet\Services\WdNisSvc" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\System\CurrentControlSet\Services\WinDefend" /v "Start" /t REG_DWORD /d "4" /f
reg add "HKLM\System\CurrentControlSet\Services\SecurityHealthService" /v "Start" /t REG_DWORD /d "4" /f
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/5670cb8f-afe4-41a5-83b9-f47e0767d39f)

- HKLM\System\CurrentControlSet\Services\WdBoot : Windows Defender Boot Driver
- HKLM\System\CurrentControlSet\Services\WdFilter : Windows Defender Filter Driver
- HKLM\System\CurrentControlSet\Services\WdNisDrv : Windows Defender Network Scan Driver
- HKLM\System\CurrentControlSet\Services\WdNisSvc : Windows Defender Network Inspection Service
- HKLM\System\CurrentControlSet\Services\WinDefend : Windows Defender service
- HKLM\System\CurrentControlSet\Services\SecurityHealthService : Windows Security Health Service

<br>

Change all services from automatic to manual<br>
When value 4 is changed, it is changed manually.<br>






