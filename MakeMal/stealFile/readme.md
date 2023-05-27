# User_stealFile

Produced IDA8.0 code impersonating IDA analysis program.<br>

When the infected file is executed, it steals information by copying documents, photos, and files from all paths of the user.<br>

In particular, Windows Defender is turned on, but it can be run.<br>

In addition, the malicious activity log was deleted so that it could not be traced.<br>

This is simply information leaking malware. It does not tamper like other ransomware, nor does socket communication take place.<br>

<br>

# video

![2023-05-18 01-38-59](https://github.com/Kwhitebear/User_stealFile/assets/99308681/de974134-bba8-40ef-9a81-953aa2540a6c)

<br>

# result

![image](https://github.com/Kwhitebear/User_stealFile/assets/99308681/04f34360-e94a-4814-9558-fcac9b13aab4)

In the test Windows, there were few files, so the transfer was fast, but even in an environment with many files, the file size is divided and transferred.<br>
