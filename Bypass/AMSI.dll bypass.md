# AMSI.dll

ASMI(Antimalware Scan Interface) is a multi-purpose interface that helps applicaations or services integrate with antivirus products.<br>

The Technology can be used in a variety of situations. For example, it can be used when a developer wants to link with an Anti-Virus product in his or her own program,<br>
and a third party can be developed and used to obtain fileless malware contents from an Anti-Virus product.<br>
Besides this, the uses are endless.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/03475f26-d50b-4b70-a652-c9b7eedc7101)

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/d64b9780-dbe1-45c3-988d-3de89b083ee3)

AMSI acts as a bridge between consumers(application programs, services, etc.) and providers (virus companies, etc.)<br>

AMSI has three componetns.<br>

1. Consumer - A component that "consumes" AMSI and can request specific data scans from AV/EDR solutions.
2. Amsi.dll - A library containing AMSI functions. Loaded into the Consumer process. Consumer Processes send AMSI requests to providers using function in this library.
3. Provider - The component that "provides" AMSI, AV/EDR solutions. After performing the scan request received through AMSI, the result is returned to the Consumer.


Originally, the consumer of AMSI was about PowerShell, but as the version goes up, all of PowerShell, JavaScript, VBA, WMI, and .NET use AMSI. AMSI Provider has been deferred to AV/EDR solution companies such as Microsoft, Crowd Strike, and Bitdefender.
AMSI is loaded into the AMSI consumer process in the form of amsi.dll library. Consumers detect malicious codes in data by sending AMSI requests to providers using functions exported in this loaded library.<br>


