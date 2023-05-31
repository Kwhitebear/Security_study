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









