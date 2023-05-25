# Winlogon.exe

# disable Windows Defender 

stop Windows Defender Programmatically creating a new token using TrustedInstaller and Windows Defender service accounts.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/c0ea4cbc-161c-49a5-96ab-9f3dcf1b09c1)


# TrustedInstaller

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/18c6b01d-84c0-436a-b670-5ba5b38cd457)

tried to delete it, but the owner of the file(TrustedInstaller), even if I was SYSTEM or administrator, prevented the deletion.

"TrustedInstaller" organizes (service groups) into virtual groups created by the Service Control Manager(SCM) at computer startup<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/d821d0b5-712a-4403-90b1-5b21bf4fc83d)

To distinguish real groups/users from SCM-created virtual groups, you need to look at the prefix "domain" that appears before users/groups.

Actuals are usually prefixed with "NT AUTHORITY" and service compounds are prefixed with "NT SERVICE"<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/d9e97022-63f3-4198-9b22-55918611f72a)



![image](https://github.com/Kwhitebear/Security_study/assets/99308681/a868739d-bcbb-4a6c-9890-851c7f456b71)

