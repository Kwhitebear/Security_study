# Winlogon.exe

# disable Windows Defender 

Windows Defender 중지 TrustedInstaller 및 Windows Defender 서비스 계정을 사용하여 새 토큰을 생성

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/c0ea4cbc-161c-49a5-96ab-9f3dcf1b09c1)


# TrustedInstaller

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/18c6b01d-84c0-436a-b670-5ba5b38cd457)

삭제를 시도했지만 파일의 소유자(TrustedInstaller)가 SYSTEM이나 관리자이더라도 삭제를 막는다.

"TrustedInstaller"는 (서비스 그룹)을 컴퓨터 시작 시 서비스 제어 관리자(SCM)가 만든 가상 그룹으로 구성<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/d821d0b5-712a-4403-90b1-5b21bf4fc83d)

실제 그룹/사용자를 SCM에서 만든 가상 그룹과 구별하려면 사용자/그룹 앞에 표시되는 접두사 "도메인"을 살펴봐야 한다.

실제는 일반적으로 "NT AUTHORITY"라는 접두사가 붙고 서비스 합성어는 "NT SERVICE"라는 접두사가 붙는다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/d9e97022-63f3-4198-9b22-55918611f72a)


아래는 winlogon.exe 관리자 Context에서 SYSTEM 액세스 토큰을 impersonation 하는데 활용한 기술이다.<br>

SYSTEM 액세스 토큰을 impersonation 하는 것은 그룹 정책을 통해 로컬 관리자 계정에서 특정 권한이 박탈된 경우에 유용하다. 예를 들어 로컬 관리자 그룹에서 "SeDebugPrivilege"를 제거하여 공격자가 자격 증명을 덤프하거나 다른 프로세스의 메모리와 상호 작용하는 것을 어렵게 만들 수 있다. 그러나 운영 체제를 실행하는데 필요한 SYSTEM 계정에서 권한을 취소할 수 없다. 그래서 이러한 환경은 공격자에게 유리하게 만든다.<br>

<h3>액세스 토큰 Steal</h3>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/a52dc779-0b35-4820-83da-08830d281609)

Windows API 호출을 사용하여 액세스 토큰을 훔칠 수 있다.<br>
- OpenProcess()<br>
- OpenProcessToken()<br>
- ImpersonateLoggedOnUser()<br>
- DuplicateToken()<br>
- CreateProcessWithTokenW()<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/edca0e0f-e31c-4a7e-acf8-ced473e8978e)

OpenProcess함수는 프로세스 식별자(PID)를 가져와서 프로세스 핸들을 반환한다. PROCESS_QUERY_INFORMATION 프로세스 핸들은 PROCESS_QUERY_LIMITED_INFORMATION 또는 PROCESS_ALL_ACCESS 사용하여 액세스 권한을 얻어야한다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/ad259d2c-a7cf-4ab8-b354-d88278766bfc)

OpenProcessToken함수는 프로세스 핸들과 액세스 권한 플래그를 입력으로 사용한다. 프로세스와 관련된 액세스 토큰에 대한 핸들을 연다. TOKEN_QUERY 및 액세스 권한으로 토큰 핸들을 열어야하거나, 사용할 수 있는 액세스 권한 으로만 토큰 핸들을 열 수 있다.(TOEKN_DUPLICATE ImperonateLoggedOnUser(), TOEKN_DUPLICATE DuplicateTokenEx())<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/2575b0c9-3ed6-483a-a4dd-95c68ee1db31)

현재 스레드가 다른 로그온 사용자를 가장하도록 허용 OpenProcessToken() 할 수 있다. 스레드 호출되거나 스레드 종료 될 때 ImpersonatedLoggedOnUser()함수를 사용하여 로그온한 사용자를 계속 impersonation 한다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/0c4a77f5-dfe4-4d20-a693-9d945fc6820d)

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/ab9af5d1-ac87-48ff-ae35-4d09966115d6)


프로세스를 다른 사용자로 생성하려면 DuplicateTokenEx() 결과 토큰 핸들을 사용하여 OPenProcessToken() 새 액세스 토큰을 만들어야 한다.<br>

<strong>순서는 trustedinstaller service open & start → get trustedinstaller pid, threadID → impersonation with thread To Using trustedinstaller’s fisrt thread → get trustedinstaller token → WinDefend off</strong><br>

윈도우 계정 중에 trustedInstaller token으로 impersonation 해서 SYSTEM 토큰으로 가장한다. 그리고 lsass.exe의 Token을 가져오고, 해당 Token으로 impersonation을 진행한다. SeCreateTokenPrivilege과 lsass.exe의 Token 바탕으로 새로운 Token 생성한다. 만들어진 Token은 "NT SERVICE\WinDefend\NTSERVICE\TrustedInstaller" 유저가 된다. 그래서 WinDefend 서비스를 관리하는 TrustedInstaller 권한을 획득하고, WinDefend를 종료 시킬 수 있다. 하지만 단점으로는 SYSTEM 알람으로 Windows Defender가 종료되었다는 알림창이 우측 하단에 팝업되는데 이를 해결해야 할것이다.


# elevate Code

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/9e68be13-4826-4cbd-8340-3b5f94ade110)

Windows 운영 체제에서 실행되는 프로세스 중 하나인 "explorer.exe"는 사용자의 작업 표시줄, 시작 메뉴 및 탐색기 등과 같은 시스템 UI를 담당한다. 이는 사용자가 Windows에서 파일 및 폴더를 찾고 관리하는 데 필수적인 기능을 제공한다. "whitelist"는 특정 프로그램이나 프로세스를 인증하고, 다른 애플리케이션들에 비해 우선적으로 실행되도록 하는 제어 목록이다. explorer.exe는 Windows 운영 체제에서 중요한 역할을 수행하기 때문에 시스템 안정성을 유지하기 위해 프로세스와 그 하위 프로세스들은 whitelist에 포함이 된다. 또한 많은 응용 프로그램들은 explorer.exe의 상호 작용하고 의존한다. 이러한 응용 프로그램들이 올바르게 동작하려면 explorer.exe와 그 하위 프로세스들이 실행되어야한다. 따라서 이러한 호환성을 보장하기 위해 whitelist 에 포함이 된다.

즉 위에 코드에서 먼저 explorer.exe 생성 하고 그에 맞게 Code Injection을 진행 한다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/a148df99-934a-49f8-af30-2a701797e459)

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/e8143900-1e4a-4626-bed7-da6c0e6ce3a5)

가상 공간을 할당하고 WriteProcessMemory 함수를 통해 값을 넣는다.(주소와 문자열들이 입력되었다.)

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/b113eafb-3980-47ac-8aba-5a03fc0f71c1)


![image](https://github.com/Kwhitebear/Security_study/assets/99308681/669ec6e3-0532-408f-9a5a-ad822a204fc7)

UAC Bypass 값을 넣는다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/389c4ce9-c8c8-4b63-807c-942225d93e4d)

마지막에는 fodhelper.exe 실행을 하면서 레지스트리에 쓰여진 값이 실행이 되면서 관리자 권한으로 실행이 된다.


참조 :
https://cinrueom.tistory.com/51
https://aidencom.tistory.com/661
https://jeep-shoes.tistory.com/11









