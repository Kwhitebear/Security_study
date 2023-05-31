# user mode hooking bypass

샌드박스(Sandbox) 기반의 보안 솔루션이나 안티바이러스(Anti-virus,백신) 제품의 행위 탐지 엔진은 악성코드의 행위를 토대로 악성 여부를 판단하고 탐지한다.<br>
유저 모드 후킹(User-Mode Hooking) 기법은 이러한 악성코드의 행위를 파악하기 위한 대표적인 기법 중 하나이다.<br>
악성코드가 실행될 때 행위에 대한 모니터링을 담당하는 DLL Injection하며, 해당 악성코드 내부에 Injection된 모니터링 DLL은 악성 행위에 필요한 주요 API 함수들을 후킹한다.<br>
이후 악성코드가 주요 API들을 호출하면 모니터링 DLL에서는 호출된 API들을 로그로 남겨 악성 행위를 판단하고 분석가가 생성한 룰을 통해 탐지까지 진행할 수 있다.<br>

특징적인 점은 유저 모드 후킹 시 주로 ntdll.dll 에서 제공하는 Native API 대상으로 한다는 점이다. 대부분의 악성코드는 악성 행위를 위해 프로세스나 메모리 혹은 파일 입출력과 관련된 자원을 사용하는데,<br>
이 자원을 사용하기 위한 API들 중 대다수는 최종적으로 ntdll.dll을 통해 시스템 콜을 호출하기 때문이다.<br>

최근 악성코드 제작자는 이러한 유저 모드 후킹 방식을 사용하는 보안 제품들을 우회하기 위해 점차 고도화된 방식의 유저 모드 후킹 우회 기법들을 개발하여 이를 악용하고 있다.<br>

# 유저 모드 후킹 우회 기법 개요

유저 모드 후킹 우회 기법은 ntdll의 후킹 여부를 검사하는 방식과 ntdll을 재로드하는 방식, 시스템 콜을 직접 호출하는 방식이 있다.<br>

먼저 ntdll의 후킹 여부를 검사하는 방식의 경우, 악성코드는 시스템 폴더의 ntdll.dll을 메모리에 읽어와 프로세스 실행 시 로드된 ntdll과 코드를 비교하며 차이가 있는지 확인한다.<br>
만약 코드에 차이가 있다면 후킹이 됐다고 판단하여 로드된 ntdll의 코드를 원본 코드로 원복시켜 후킹을 우회한다.<br>

ntdll 재로드하는 방식은 프로세스 내부에 ntdll.dll을 또 로드하여 프로세스 실행 시 로드된 기존 ntdll.dll의 API를 호출하지 않는 방법이다.<br>
기본적인 개념으로는 같은 dll을 재로드 할 수 없으나 간단한 트릭을 이용해 같은 dll을 재로드 시킬수 있다.<br>

마지막으로 직접 시스템 콜을 호출하는 방식이 있다. ntdll.dll의 Native API는 커널에 자원을 요청할 경우 해당 기능에 맞는 번호와 함께 시스템 콜을 호출하는데, 악성코드 역시 해당 기능에 맞는 번호를 구성하고<br>
시스템 콜을 호출하여 후킹 대상인 ntdll을 거치지 않고 목적을 달성할 수 있다.<br>

https://image.ahnlab.com/file_upload/asecissue_files/ASEC%20REPORT_vol.97.pdf

# More

2017년 파워쉘의 몰락을 거쳐 .NET/C#/C++/Nim/Golang등을 이용해 로우-레벨의 툴 제작(Tradecraft)을 시도하고 있다. 이에 발 맞춰 2013년도에 EDR 솔루션이 개발됐고, 2016년 부터 본격적으로 EDR 솔루션을 도입하는 회사들이 많아짐에 따라 EDR vs 공격자 구도가 형성됐다.<br>

2021년 기준 많은 EDR 솔루션들이 커널 모드로 다시 돌아가며 다양한 탐지 방법을 만들어내고 있지만, 그 전에 가장 많이 쓰이던 탐지 기법은 user mode 후킹이다.<br>

# 앤드포인트 탐지 트렌드

2021년 기준 현재 시장에 나와있는 EDR 솔루션들은 다양한 방법을 통해 악성코드 및 악성행위를 탐지한다. 커널 접근이 가능했던 "옛날"과 달리, Kernel Patch Protection(PatchGuard)이 적용된 이후 보안솔루션들은 커널 모드에서 벗어나 유저 모드에서 탐지를 진행해야 한다.<br>
유저 모드에 있는 탐지 방법 및 데이터 측정 출저(Telemetry sources)의 종류는 많이 있지만, 가장 많이 알려져 있는 기법/기술은 아래와 같다.<br>
- user mode 후킹(Userland Hooking)
- 커널 콜백 함수(Kernel Callbacks)
- ETW(Event Tracing for Windows)
- 미니필터 드라이버(Mini-Filter Drivers)
- 파워쉘과 .NET AMSI(AntiMalware-Scanning-Interface)


![image](https://github.com/Kwhitebear/Security_study/assets/99308681/a1e71a36-6bff-406a-bc00-3426ede194ed)

윈도우 운영체제는 크게 유저 모드(유저 랜드)와 커널 모드(커널 랜드)로 나뉘어져있다. 이를 링3, 링0이라고도 부른다. 유저 모드에서는 프로세스가 실행되며, 커널 모드에서는 하드웨어와 소프트웨어를 이어주는 커널이 존재한다. 커널은 운영체제의 핵심이기 때문에 잘못된 접근을 할 경우 머신 전체가 장악당하거나 잘못될 위험(크래시,블루 스크린)이 있다. 이 때문에 링0과 링3은 분리 되어있다.<br>

하지만 유저 모드가 커널에 접근해야하는 경우도 생기기 마련이다. 예를 들어 유저 모드의 프로세스인 메모장을 이용하다가 파일 저장을 하고 싶으면, 커널을 이용해 하드웨어인 디스크에 파일을 저장해야한다.<br>

예를 들어, 다음과 같은 Python Code 실행한다.

```
fd = open("test.txt", "w")
fd.write("creating and writing")
fd.close()
```

이 코드는 유저 모드에 존재 하지만, 파일 시스템을 통해 하드웨어에 test.txt라는 파일을 생성해야 한다. 이처럼 유저 모드의 프로그램이 커널에 접근해 조작을 해야할 경우가 생긴다. 이 대 유저 모드와 커널 모드를 이어주는 것이 바로 윈도우 API(winapi, win32api, windows API)다. 윈도우 API는 웹개발을 할 때 자주 보던 API와 비슷한 개념이다. 유저 모드의 프로그램이 커널과 소통을 해야할 경우가 생길 때 윈도우 API를 이용하면 정해진 규칙과 규정대로 커널에 접근할 수 있다.<br>

윈도우 API 함수들은 kernel32.dll, kernelbase.dll, ntdll.dll 과 같은 다양한 라이브러리 파일들의 export table 에 존재한다. 이 중 kernel32.dll, kernelbase.dll, ntdll.dll 과 같은 라이브러리들은  프로세스 실행시 프로세스의 메모리로 기본적으로 로드된다. .exe, 메모장, 어떤 실행파일을 실행하던간에 기본적으로 로드된다.<br>


![image](https://github.com/Kwhitebear/Security_study/assets/99308681/464be99a-54d8-42d1-bc53-8f85a3a69452)

CreateFile과 WriteFile이 보인다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/3a7152ee-3362-45ee-b352-667cb75e89fa)

자세히보면 NTCreateFile이 보인다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/b045b415-dac2-494c-8bd0-0154399ce29e)

마찬가지 NtWriteFile이 보인다.<br>

ntoskrnl.exe는 Windows의 핵심 서비스와 기능을 제공하며, 운영 체제의 안정성과 보안을 유지하는 데 중요한 역할을 담당한다. 이 파일은 메모리 관리, 프로세스 및 스레드 관리, 디스크 및 파일 시스템 관리, 하드웨어 추상화, 보안 등 다양한 시스템 기능을 처리한다.(커널 모드에서 실행되는 핵심 컴퓨터 프로그램)<br>

각각의 CreateFile 과 WriteFile를 보면
- CreateFile = Python의 "Open('test.txt','W')" -> CreateFileW(kernelbase.dll) -> NtCreateFile(ntdll.dll) -> NtCreateFile(ntoskrnl.exe)
- WriteFile = Python의 "fd.write" → WriteFile (kernelbase.dll) → ZwWriteFile (ntdll.dll) → NtWriteFile (ntoskrnl.exe)


유저 모드(U)에서 커널 모드(K)로, 아래서 위로 함수들이 실행된다. 윈도우 API 함수를 실행하면 kernelbase.dll의 함수 호출 -> ntdll.dll 함수 호출 -> ntoskrnl.exe 커널에서 시스템 콜 실행과 같은 방법으로 실행되고 있다. 마지막으로, 똑같은 CreateFileW가 아니라 ntdll.dll 과 ntoskrnl.exe에서는 앞에 "Nt" 접두어가 붙은 Native API가 실행된다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/0693e61a-b5b4-4c60-8bb3-208b1011b53d)

사용되는 라이브러리들을 위에서 아래로(유저모드 -> 커널모드)로 정리해보면 아래와 같다.
1. Kernel32.dll = 메모리 I/O 관련 윈도우 API를 가지고 있다. 단, 윈도우7/서버2008 윈도우 버전에서는 kernelbase.dll로 코드 실행을 넘기는 역할을 담당한다.
2. Kernelbase.dll = 최근 윈도우 버전부터 kernel32.dll로부터 인자를 받고 실질적으로 관련 윈도우 API를 가지고 있다. 커널베이스 또한 실질적으로 ntdll.dll의 네이티브 API를 실행하는 역할 담당이다.
3. Ntdll.dll = 윈도우 API보다 한단계 낮은 Native API 가지고 있다. 윈도우 API는 Native API의 래퍼(wrapper,포장지개념)일 뿐이다.Native API는 공식적으로 문서화 되어있지 않고, 윈도우 버전이 바뀔때마다 바뀌는 경우가 많아 이를 추상화 시키기 위해 윈도우 API가 사용 된다. Ntdll.dll의 Native API를 통해 최종적으로 시스템 콜 CPU 명령어가 어셈블리 단계에서 실행 된다.
4. Ntoskrnl.exe = 코드 실행은 커널로 넘어가 System Service Dispatch Table(SSDT)를 이용해 시스템 콜 번호를 찾아 실행 한다.

<h2>System Service Dispath Table(SSDT)</h2>

SSDT(System Service Dispatch Table)는 시스템 호출을 요청한 뒤, 전달되는 서비스 번호에 맞는 함수를 찾을 때 참조한다.<br>

시스템 호출을 요청하는 두 가지 명령어(INT 0x2E와 SYSENTER)가 KiSystem Service(system service Dispatcher)를 호출한다.
1. INT 0x2E나 SYSENTER 같은 시스템 콜이 호출된다. (XP 이후는 SYSENTER를 사용)
  - http://luckey.tistory.com/86<br>
2. 시스템 콜이 호출되면 커널에서 KiSystemService(시스템 서비스 디스패처)를 호출한다.
  - EAX에서 시스템 콜 번호를 읽어 SSDT의 인덱스로 사용<br>
  - EDX는 인자를 유저모드 스택에서 커털 모드 스택으로 복사한 곳을 가리킴<br>
3. EAX에서 받은 시스템 콜 번호에 맞게 KeServiceDescriptorTable(SSDT)를 참조하여 Native API를 호출한다. 
  - `[SSDT+EAX*4]와 같이 참조<br>
4. 시스템 콜 종료하고 유저 모드로 복귀한다. 
  - INT 0x2E의 경우 iretd(인터럽트 복귀 명령) 사용.<br>
  - SYSENTER의 경우 SYSEXIT 사용.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/c491b43b-1e74-4e08-a495-52b7f30eef75)

KiSystemService가 호출될 때 EAX에는 사용자 영역에서 요청한 서비스 번호가 저장되어 있으며, EDX에는 이러한 서비스에 사용될 인자가 저장되어 있다. 이러한 시스템 호출 번호(EAX)에 맞게 KeServiceDescriptor Table에서 Native API의 주소를 가지고 온다. 그 후 시스템 호출을 종료하고 다시 사용자 모드로 복귀하게 된다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/5cd71c25-76f5-4e3b-9375-f2f14d2e25d0)

SSDT(KiServiceTable)에서 서비스 호출 번호에 맞는 주소를 얻은 다음 이를 호출하는 형태로 진행 되는 것이다. 그렇다면 SSDT Hooking은 어느 부분을 후킹해야 하냐면 바로 GetfuncAddress과정에서 후킹한다고 할 수 있다. SYSENTER를 통해 KiFastCallEntry로 진입한 후 서비스 번호에 맞는 서비스 루틴을 SSDT에서 얻어온다. 따라서 SSDT에 존재하고 있는 각 서비스 루틴의 주소를 조작하므로 후킹을 진행할 수 있다.<br>

해당 시스템 호출의 서비스 루틴을 가지고 오는 과정에서 SSDT를 참조하는데, SSDT의 해당 번호가 나타내는 주소를 후킹하므로 우리가 원하는 흐름으로 조작할 수 있다. 위 그림을 예로 시스템 호출이 요청되었을 때 서비스 번호가 저장되어 있는 EAX의 값이 0xAD라면 SSDT에서 0xAD가 가리키는 서비스 루틴의 주소 0xCCCCCCCC가 반환되어 이를 호출한다. 하지만 만약 공격자가 SSDT를 후킹하여 0xDDDDDDDD로 서비스 루틴의 주소를 변경하였다면, 시스템 호출 0xAD가 발생할 때마다 0xDDDDDDDD를 지나가게 된다.<br>

# Hooking 후킹

다시 돌아와서 User mode hooking에 대해 알아본다.<br>

후킹은 특정 시스템의 함수 콜/ 메시지 / 이벤트 등을 가로채 그 시스템의 행동을 바꾸는 기법들을 일컫는다. 자신의 프로세스에 로드된 kernel32.dll 등의 라이브러리를 통해 윈도우 API를 사용한다. 따라서 프로세스에 로드된 kernel32.dll의 윈도우 API를 후킹하면 악성 행위를 잡을 수 있다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/e6893954-fb00-40b5-84c7-b744fa725e38)

예를 들어 악성코드 실행시 "C:\Users\malware.txt" 라는 파일을 생성한다고 가정해보면
1. 생성되는 모든 프로세스의 kernel32.dll 라이브러리의 CreateFileW() 윈도우 API를 후킹한다.
2. 악성코드가 kernel32.dll의 CreateFileW() 윈도우 API를 이용한다.
3. IF ) Kernel32.dll의 CreateFileW()함수는 후킹되었기 때문에 실행시 방어자의 프로세스로 가로챈다. 방어자의 프로세스는 매개변수 중 "C:\Users\malware.txt를 발견한 뒤, 악성코드라고 판단, 타겟 프로세스를 종료시킨다.
4. ELSE ) 정상 프로그램 후킹 후 CreateFileW()의 매개변수 중 이상한 점을 발견하지 못했다. 악성코드가 아니라고 판단, 다시 실행을 kernel32.dll로 넘겨준다. 그 뒤 CreateFileW() 함수는 정상적으로 실행된다.

EDR 솔루션들이 사용하는 후킹 방법중 하나는 DLL 인젝션을 이용한 인라인 후킹 (inline hooking)이다.

인라인 후킹은 타겟 함수의 주소를 찾은 뒤, 첫 n바이트의 어셈블리 명령어들을 JMP등의 명령어로 패치해 코드실행을 가로채간다. 예를 들어 이번에는 ntdll.dll 라이브러리에 있는 "NtProtectVirtualMemory" 윈도우 API를 패치한다고 가정하면, 해당 함수는 악성코드들이 쉘코드를 메모리에 작성/옮겨넣기 전, 메모리의 권한을 바꾸는데 자주 사용되는 함수이다.<br>


NtProtectVirtualMemory : 'Virtual Protect' API 내부 호출 API(호출 프로세스의 가상 주소 공간에서 커밋된 페이지 영역에 대한 보호를 변경)<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/fc17a979-bf3c-445d-9a96-b45a886c90e3)

정상적인 NtProtectVIrutalMemory 이다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/060db73a-2214-4089-9de0-3ee299e48789)

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/4877e311-c9fa-45b1-9857-2951453a0341)

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/71edb7a7-e2ea-4dff-a2e8-1a17a6708f8d)

DLL Injection 상태이다.

아래는 Injection 

```
#define _CRT_SEUCRE_NO_WARNINGS

#include <stdio.h>
#include <Windows.h>
#include <TlHelp32.h>
#include <tchar.h>

int main(int argc, char* argv[])
{
    DWORD pid = 0;
    LPVOID DllBuf;
    HANDLE hSnapshot, hProcess, hThread;
    HMODULE hMod;
    LPTHREAD_START_ROUTINE hModAddr;
    LPCTSTR DllPath = "C:\\Project1.dll";

    PROCESSENTRY32 pe32 = { 0, };
    pe32.dwSize = sizeof(PROCESSENTRY32);

    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPALL, NULL);
    if (Process32First(hSnapshot, &pe32))
    {
        do
        {
            if (!_tcsicmp(pe32.szExeFile, TEXT("notepad.exe")))
            {
                printf("[+] Find Process\n");
                pid = pe32.th32ProcessID;
                printf("[+] PID : %d\n", pid);
                break;
            }
        } while (Process32Next(hSnapshot, &pe32));
    }CloseHandle(hSnapshot);

    if (!(hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid)))
    {
        printf("[-] OpenProcess Fail\n");
    }
    printf("[+] OpenProcess Success 0x%x\n", hProcess);

    if (!(DllBuf = VirtualAllocEx(hProcess, NULL, lstrlen(DllPath) + 1, MEM_COMMIT, PAGE_READWRITE)))
    {
        printf("[-] VirtualAllocEx Fail\n");
    }
    printf("[+] VirtualAllocEx Success 0x%x\n", DllBuf);

    if (!(WriteProcessMemory(hProcess, DllBuf, (LPVOID)DllPath, lstrlen(DllPath) + 1, NULL)))
    {
        printf("[-] WriteProcessMemory Fail\n");
    }

    if (!(hMod = GetModuleHandle(TEXT("kernel32.dll"))))
    {
        printf("[-] GetModuleHandle Fail\n");
    }
    printf("[+] kernel32.dll : 0x%x\n", hMod);

    if (!(hModAddr = (LPTHREAD_START_ROUTINE)GetProcAddress(hMod, "LoadLibraryA")))
    {
        printf("[-] GetProcAddress Fail\n");
    }
    printf("[+] Kernel32.LoadLibarayA : 0x%x\n",hModAddr);

    if (!(hThread = CreateRemoteThread(hProcess, NULL, 0, hModAddr, DllBuf, 0, NULL)))
    {
        printf("[-] CreateRemoteThread Fail\n");
    }
    printf("[+] Thread Handle : 0x%x\n", hThread);
    WaitForSingleObject(hThread, INFINITE);
    CloseHandle(hThread);
    CloseHandle(hProcess);
    system("pause");

    return TRUE;
}
```


![image](https://github.com/Kwhitebear/Security_study/assets/99308681/2b907bb6-db6d-4fbf-a5a0-4b5414b9b416)

본 환경에서는 아무리 분석해도 계속 실행이 잘된다. 그래서 다른 블로그를 참조하면

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/482b1032-a406-4dcf-9158-177201db1937)

메시지 박스가 출력이되면서 막아버린다고 한다.

후킹된 결과는 아래의 블로그를 확인해보자

https://blog.sunggwanchoi.com/kr-yujeo-raendeu-huking/














