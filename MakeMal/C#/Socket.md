# Socket

It is possible to steal victim information, transfer files, execute, and input commands while communicating through sockets.
<br><br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/cf5b53e1-745b-4cb8-a28a-349de0d9b87f)

Click the Server and Client Download button to download the executable file.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/ec1b3459-3d17-4f89-a6fd-ade68cfc8cab)

If check the folder, socket executable files have been created.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/a12dd1a9-1acf-4766-909d-76dabc95f4de)

Server Send Command "cmd /c", The victim executes the command.<br><br>


![2023-05-30-13-24-35](https://github.com/Kwhitebear/Security_study/assets/99308681/a142b8a0-0bab-46f3-8451-ad2b79d6466f)

The connection may be disconnected, but the connection is still possible.<br>


![image](https://github.com/Kwhitebear/Security_study/assets/99308681/894285f9-7db5-4c3d-9395-d8b7ee7ffba2)

To run powershell, need to check the policy. Because in Windows, it is basically set to "Restricted" where execution is restricted except for Script .PS1 files(signed files) created by Microsoft.<br>


![image](https://github.com/Kwhitebear/Security_study/assets/99308681/b664117d-fb96-4112-b3b6-961717e418ac)

So, in order to execute powershell Script, need to change "Unrestricted" policy to execute all script.<br>
