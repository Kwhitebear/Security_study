# SmartScreen.exe

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/a97a6b7e-40a5-4f04-b059-000f8d3d3056)

SmartScreen.exe WinVerifyTrust 함수에서 검증을 시도한다. "WinVerifyTrust" 함수는 지정된 개체에 대한 신뢰 확인 작업을 수행하는데 인증서 확인을 위해 "CertGetCertificateChain" 및 "CertVerifyCertificateChainPolicy" 함수를 사용한다.<br>

```
LONG WinVerifyTrust(
  [in] HWND   hwnd,
  [in] GUID   *pgActionID,
  [in] LPVOID pWVTData
);
```
<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/15a31c84-538e-41c4-9e64-29c1bb37b3d8)

- CERT_CONTEXT 구조(wincrypt.h) : CERT_CONTEXT 구조에는 인증서의 인코딩과 디코딩 모두 포함이 되어 있다. Wincrypt.h에 정의된 함수 중 하나에서 반환된 인증서 CONTEXT는 CertFreeCertificateContext 함수를 호출하여 해제하여야 한다.<br>

```
typedef struct _CERT_CHAIN_CONTEXT {
  DWORD                cbSize; // 구조의 크기
  CERT_TRUST_STATUS    TrustStatus; // 체인 배열의 결합된 신뢰 상태를 나타내는 구조(오류 상태 코드와 정보 상태 코드               
  DWORD                cChain; // 배열에 있는 체인의 수
  PCERT_SIMPLE_CHAIN   *rgpChain; // 체인 구조에 대한 포인터 배열 rpgChain[0] 최종 인증서 단순 체인, rgpChain [cChain-1]최종 체인
  DWORD                cLowerQualityChainContext; // rgpLowerQualityChainContext 배열의 체인 수
  PCCERT_CHAIN_CONTEXT *rgpLowerQualityChainContext; // CERT_CHAIN_CONTEXT 구조에 대한 포인터 배열, dwFlags에 설정된 경우 반환
  BOOL                 fHasRevocationFreshnessTime; // dwRevocationFreshnessTime을 사용할 수 있는 경우 TRUE 로 설정된 부울 값
  DWORD                dwRevocationFreshnessTime; //가장 큰 CurrentTime(초)에서 확인된 모든 요소의 CRL( 인증서 해지 목록 ) ThisUpdate를 뺀 값
  DWORD                dwCreateFlags;
  GUID                 ChainId;
} CERT_CHAIN_CONTEXT, *PCERT_CHAIN_CONTEXT;
```

<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/1c505372-7870-4a92-a251-5cc284b7f296)

<strong>- PCCERT_CHAIN_CONTEXT :</strong> 인증서 체인 배열과 연결된 모든 체인에 대한 유효성 데이터를 나타내는 신뢰 상태 구조<br>

<strong>- CERT_CHAIN_POLICY_PARA :</strong> 인증서 체인 확인을 위한 정책 기준을 설정하기 위해 CertVerifyCertificateChainPolic 에서 사용되는 정보
<br>

```
typedef struct _CERT_CHAIN_POLICY_PARA {
  DWORD cbSize; // 구조의 크기
  DWORD dwFlags; // 유효하지 않을 수 있고 인증서 체인을 구축할 때 무시해야 하는 조건을 나타내는 플래그 집합
                    [=] pszPolicyOID 매개 변수는 아래의 값 중 하나를 포함
                    CERT_CHAIN_POLICY_BASE
                    CERT_CHAIN_POLICY_AUTHENTICODE
                    CERT_CHAIN_POLICY_AUTHENTICODE_TS
                    CERT_CHAIN_POLICY_SSL
                    CERT_CHAIN_POLICY_NT_AUTH         
  void  *pvExtraPolicyPara; // 추가 유효성 정책 조건을 제공하는 pszPolicyOID 관련 구조의 주소
} CERT_CHAIN_POLICY_PARA, *PCERT_CHAIN_POLICY_PARA;
```

<br>
<strong>- CERT_CHAIN_PARA :</strong> 인증서 체인을 구축하는데 사용되는 검색 및 일치 기준을 설정<br>

```
typedef struct _CERT_CHAIN_PARA {
DWORD cbSize; //구조의 크기
CERT_USAGE_MATCH RequestedUsage; // 인증서 체인을 구축하기 위해 발급자 인증서를 찾는데 필요한 일치 유형을 나타내는 RequestedUsage 구조
} CERT_CHAIN_PARA,  *PCERT_CHAIN_PARA;
```

<br>
<strong>- CERT_CHAIN_POLICY_STATUS :</strong> 인증서 체인의 유효성을 검사할 때 CertVerifyCertificateChainPolicy 함수에서 반환한 인증서 체인 상태 정보를 보유<br>

```
typedef struct _CERT_CHAIN_POLICY_STATUS {
  DWORD cbSize; // 구조의 크기
  DWORD dwError; //유효성 검사 프로세스 중에 오류 또는 잘못된 조건이 발생했음을 나타내는 값
  LONG  lChainIndex; //유효하지 않은 오류 또는 조건이 발견된 체인을 나타내는 인덱스
  LONG  lElementIndex; //잘못된 오류 또는 조건이 발견된 체인의 요소를 나타내는 인덱스
  void  *pvExtraPolicyStatus; //구조체에 대한 포인터
} CERT_CHAIN_POLICY_STATUS, *PCERT_CHAIN_POLICY_STATUS;

```
<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/5e7b8876-0e28-4cbb-b00c-d19dbe2dc9bd)

참고로 wintrust.dll 동적으로 불러오는데 wintrust.dll은 Microsoft Trust Verification API 가지고 있다.

즉, wintrust.dll은 디지털 서명을 확인하는 데 사용되는 Windows API를 제공하며 smartscreen.exe는 다운로드 된 파일의 디지털 서명을 확인하여 해당 파일이 의심되는 출저에서 확인하는데 디지털 서명이 유효하지 않은 경우 smartscreen.exe는 해당 파일을 위험하다고 판단한다. 그리고 wintrust.dll은 보안 관련 업데이트를 제공하고, 시스템의 보안 정책을 적용한다.<br>

# wintrust.dll

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/c52e0eb5-c90e-4ed0-a96b-5ebaca993c6c)

WintrustGetRegPolicyFlags 함수는 정책 공급자에 대한 정책 플래그를 검색이며 아래는 sub_180040070 내부 로직을 따라간 결과이다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/af8a5112-4edf-437e-8078-f2891926919b)

result 변수에 sub_18000A840 함수에서 무언가를 진행하고 반환된 값을 저장한다. 그리고 그 반환된 값이 없을 경우 "HCKU\\SOFTWARE\\Windows\\CurrentVersion\\WinTrust\\Trust Providers\\Software Publishing" 경로에 값을 생성한다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/31c981ae-dbc7-425b-9722-67c68ad3a9cd)

분석 중인 컴퓨터 뿐만 아니라 대부분의 컴퓨터는 위의 사진처럼 값이 설정되어 있을것이다.<br>
WTPF_IGNOREREVOCATIONONTS = 0x00020000 = 타임스탬프를 해지 확인은 무시<br>
WTPF_OFFLINEOKNBU_COM = 0x00002000 = 원본이 오프라인인 경우 상업용 인증서를 신뢰 그러나 UI 사용하면 안됨<br>
WTPF_OFFLINEOKNBU_IND = 0x00001000 = 원본이 오프라인인 경우 개별 인증서를 신뢰 그러나 UI 사용하면 안됨<br>
WTPF_OFFLINEOK_COM = 0x00000800 = 원본이 오프라인인 경우 상업용 인증서를 신뢰<br>
WTPF_OFFLINEOK_IND = 0x00000400 = 원본이 오프라인인 경우 개별 인증서를 신뢰<br>

sub_7558A840 내부를 확인해보면 아래와 같다.

 ![image](https://github.com/Kwhitebear/Security_study/assets/99308681/2aea91d8-043c-4e78-8d91-96ea11a985f8)
 
 SID 관련된 토큰을 생성하고 검증한다.
 
 

 


# Code Signing Certificate

With the digital signature process of executable files and scripts introduced by MS, these certificates prove that the files are safe.
SmartScreen filter manages its own white list database of certificates to determine if the certificate is secure or note.
judge If the certificate does not exist or if the certificate is invalid because the file has been tampered with, a warning is displayed when running. 
Notify Users.<br>

In order to bypass web browser file downloads, antivirus, and Windows Defender Smart Screen, the program needs to be signed.
This is done by purchasing a certificate from a company, signing the program with a private key, and granting the certificate. Even if a certificate is granted, Smart Screen will continue to occur until the certificate has acquired a reputation for being secure.<br>

1. Signed program distribution -> (Smart Screen occurs) -> Program report X -> Reputation improvement of the signature -> Smart Screen does no occur<br>

<strong>more imformation : https://twoicefish-secu.tistory.com/411</strong>


CVE-2023-24880 (CVSS score: 5.4), is a security feature bypass found in Windows SmartScreen, which allows to create executables that can bypass Windows Mark of the Web (MotW) security warnings. Although it has a CVSS score of 5.4, it is under active exploitation


# MsMpeng.exe Difference SmartSreen.exe

<strong>MsMpenge.exe</strong> : one of the main components of Windows Defender, which is used to scan your computer in real time and detect malware.<br>
<strong>SmartScreen.exe</strong> : Protects the computer from malicious code or threats by checking the safety of files or executable files downloaded by the user.<br>

<h3>Difference</h3>
<p>1. To do :</p>
-Smartscreen.exe is primarily responsible for evaluating the reliability of downloaded files and blocking malicious code.<br>
-MsMpenge.exe is reponsible for Windows Defender's core role of detecting and blocking malware. MsMpenge.exe scans your computer's files and system health to identify and remove malicious code.<br>
<br>
<p>2. Scope of role :</p>
-SmartScreen.exe is primarily focused on evaluating the reliability of files downloaded from the Internet.<br>
-MsMpenge.exe protects your entire system, detects malware, and provides real-time protection for files, folder, executables, and more.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/607f800e-d432-445b-bb68-98deca7b6293)

If the executable is added to the regedit path("HKLM\Software\Microsoft\Windows NT\CUrrentVersion\Image File Execution Options\") and the debugger flag is set, other programs amy run before that program runs.
If the program that runs first does not call that program, you can substitute that program. This can prevent various important programs from running.

<h3>If you want to block "smartscreen"</h3>
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\smartscreen.exe" /v Debugger /d "cmd.exe /C >null: 2>null:"<br>

<h3>If you want to block "Windows Defender"</h3>
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\securityhealthhost.exe" /v Debugger /d "cmd.exe /C >null: 2>null:"<br>
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\secHealthUI.exe" /v Debugger /d "cmd.exe /C >null: 2>null:"<br>
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\werFault.exe" /v Debugger /d "cmd.exe /C >null: 2>null:"<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/03c52498-451b-4df5-9222-526db34b7964)

Run system32









