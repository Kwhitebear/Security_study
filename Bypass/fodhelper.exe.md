# fodhelpr.exe

UAC란? 
- User Account Control의 약자로 마이크로소프트가 윈도우 비스타에서 처음 선보인 보안 기술이다. 이 기술은 응용 프로그램들의 권한을 표준 사용자 권한으로 제한을 두고 권한을 높이기 위해서는 관리자가 권한 수준을 높이는 것을 허용해야만 한다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/a1827d49-cb38-44ca-8d7b-21e9d912fa48)

<h2>Auto-elevated Executable</h2>
Auto-elevated Executable은 자동으로 권한이 상승되어 실행되는 것을 말한다. 자동으로 권한이 상승된다는 것은 UAC를 표시해 허용을 받지 않고 권한이 상승된 상태로 실행이 되는것을 말한다. 예외는 System32 폴더 내부에 저장된 프로그램은 마이크로소프트에서 신뢰하는 파일로 자동으로 권한이 상승되어 실행 된다. 또는 마이크로소프트의 인증을 받은 프로그램도 자동으로 권한이 상승되어 실행이 된다. 이 때, UAC가 나오지 않고 자동으로 관리자 권한으로 실행이 된다.

<h2>UAC Bypass</h2>
UAC Bypass는 프로그램을 관리자 권한으로 실행할 때 사용자 계정 컨트롤(UAC)를 표시하지 않고 관리자 권한으로 실행하는 기법을 말한다.

기법에는 여러가지가 있지만 fodhelper.exe 와 computerdefaults.exe 가 있다. 이 중에서 본문은 fodhelper.exe에 대해 알아본다.

<h2>Fodhelper.exe</h2>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/941e493f-19dd-4495-b964-7e1b52957ab3)

Windows 10에 존재하는 선택적 기능을 활성화하거나 비활성화 하는 설정이다.
fodhelper.exe를 실행하면 Windows 10의 설정이 실행되면서 자동으로 선택적 기능을 설정하는 창으로 이동된다. 하지만 이 때, fodhelper.exe를 실행시킬 때 Auto-elevated Executable이 적용되어 자동으로 관리자 권한으로 실행된다. 그 후 fodhelper.exe가 관리자 권한 상태에서 설정을 실행시키는데 이를 이용해 UAC Bypass 수행 할 수 있다. UAC Bypass 시도하기 위해서는 아래의 경로 레지르스티를 수정하면 된다.

```
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\ms-settings\Shell\Open\Command\(default)
또는
HKEY_CURRENT_USER\SOFTWARE\Classes\ms-settings\Shell\Open\Command\(default)
```

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/d1467606-903d-461a-9711-c5c34e4e44ae)

레지스트리를 확인해보면 위와 같이 설정이 되어 있는 것을 볼 수 있는데 기본값(default)에 UAC Bypass를 하기 위한 프로그램의 경로를 넣어주고 DelegateExecute의 값을 지운다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/49499ba3-bc96-4891-9531-995c5f2e7b3f)

설정 후 fodhelper.exe 실행시키면 선택적 기능 설정은 실행되지 않고 cmd.exe가 관리자 권한으로 실행되는 것을 확인 할 수 있다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/7834b85b-6b7a-4918-8bbd-02d6cc4dac7a)

fodhelper.exe를 이용해 UAC Bypass가 가능한 이유는 다음과 같은 이유 때문인데 먼저 fodhelper.exe가 Auto-elevated Executable로 UAC 표시 없이 관리자 권한으로 실행 되기 때문이다. 그 상태에서 cmd.exe를 실행하기 때문에 fodhelper.exe의 자식 프로세스로 UAC 표시 없이 관리자 권한으로 실행이 된다. 

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/ea43cac3-1ec3-41ce-9cef-76a425205b77)

fodhelper.exe의 코드를 보면 ShellExecuteExW 함수를 사용해 무언가를 Open 하는 것이 보인다. pExecInfo는 SHELLEXECUTEINFOW 구조체로 shellExecuteExW 함수의 인자로 들어간다. SHELLEXECUTEINFOW 구조체에는 shellExecuteExW 함수가 어떤 일을 수행 하는지에 대한 정보가 담겨져 있는데 여기서 중요한건 "IpVerb" 와 "IpFile" 이다. 우선 pExecInfo.IpVerb는 수행할 작업을 지정하는 문자열이다. 코드를 보면 Open이라고 되어 있다.
- Open : IpFile 매개변수로 지정된 파일을 연다.  파일은 실행 파일, 문서 파일 또는 폴더 일 수도 있다.

그 다음은 pExecInfo.IpFile인데 이 부분이 중요하다. 지정된 파일이 ms-settings:optionalfeatures 라고 되어 있는데 이 부분에 대해서는 아직 공식 문서를 찾지 못했다. 하지만 레지스트리 경로는 보았을 때 ms-settings는 컴퓨터 \HKEY_LOCAL_MACHINE\SOFTWARE\Classes\(name)\shell\Open\Command\(deafult)에서 name에 해당하는 것으로 보인다. 그런데 여기서 optionalfeatures가 무엇을 의미하는지 찾지 못했기 때문에 직접 코딩을 하여 확인을 해봤다.

```
#include <stdio.h>
#include <windows.h>

int main()
{
    SHELLEXECUTEINFOW pExecInfo;
    memset(&pExecInfo, 0, 0x70);
    pExecInfo.hwnd = 0;
    pExecInfo.lpVerb = L"open";
    pExecInfo.cbSize = 0x70; // sizeof(SHELLEXECUTEINFOW);
    pExecInfo.lpFile = L"ms-settings:optionalfeatures";
    pExecInfo.fMask = 0x500; // SEE_MASK_FLAG_NO_UI | SEE_MASK_FLAG_DDEWAIT
    pExecInfo.nShow = 1;
    WINBOOL bSuccess = ShellExecuteExW(&pExecInfo);
    if (!bSuccess)
    {
        printf("GetLastError : %d\n", GetLastError());
    }
}
```

코드를 컴파일 하여 관리자 권한으로 실행 시켜보면 레지스트리 경로에 설정 했던 프로그램이 실행 하게 된다. 그런데 여기서 코드를 아래와 같이 수정한다.

```
#include <stdio.h>
#include <windows.h>

int main()
{
    SHELLEXECUTEINFOW pExecInfo;
    memset(&pExecInfo, 0, 0x70);
    pExecInfo.hwnd = 0;
    pExecInfo.lpVerb = L"open";
    pExecInfo.cbSize = 0x70; // sizeof(SHELLEXECUTEINFOW);
    pExecInfo.lpFile = L"ms-settings";
    pExecInfo.fMask = 0x500; // SEE_MASK_FLAG_NO_UI | SEE_MASK_FLAG_DDEWAIT
    pExecInfo.nShow = 1;
    WINBOOL bSuccess = ShellExecuteExW(&pExecInfo);
    if (!bSuccess)
    {
        printf("GetLastError : %d\n", GetLastError());
    }
}
```

"ms-settings:optionalfeatures"에서 ":optionalfeatures"를 지우면 프로그램이 실행이 되지 않는다.
추측을 해보면 RegistryKey:Optionalfeatures과 같은 형식으로 IpFile을 설정 후 IpVerb가 open인 경우 레지스트리에서 같은 이름의 키를 찾은 후 \HKLM\SOFTWARE\Classes\(RegistryKey)\shell\Open\command\의 DelegateExecute에 값이 존재 하지 않는다면 (default)의 값에 있는 프로그램이 실행하는 것으로 추측이 된다. 그 외에 다른 System32 내부에 있는 프로그램에서 다음과 같은 코드가 존재하는 프로그램이라면 UAC Bypass가 일어날 수 있다고 추측이 된다.

참고로 현재는 레지스트리 경로에 cmd.exe 및 cmd.exe 통한 powershell.exe 값이 설정이 되어있으면 윈도우 디펜더에서 탐지하여 설정하지 못하게 하고 있다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/f4217d61-d0ab-4806-9c9b-cf5a68b6ec29)

x64dbg 디버깅을 하는데 IDA 정적 분석이랑 별 차이는 없다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/4f923120-75ce-4a93-bed2-7cc98e182fea)

바이너리 변조를 하여 "ms-settings" 뒤에 문자열들을 제거하고 실행하면 실행이 되지 않는다.


# 참조

https://learn.microsoft.com/en-us/windows/win32/api/shellapi/ns-shellapi-shellexecuteinfow
