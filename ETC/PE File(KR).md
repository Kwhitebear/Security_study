# PE File

악성 코드 분석뿐만 아니라 기본적으로 알아야할 지식 공부.<br>

PE(Portable Executable) 파일은 Windows 운영체제에서 사용되는 실행 파일 형식이다. 기존 UNIX의 COEF(Common Object File Format)을 기반으로 Microsoft에서 만들었으며, Windows 운영체제에서만 사용된다.<br>
- EXE, SCR, SYS, dll
- PE 파일 실행 -> 헤더 정보를 메모리에 매핑 -> 실제 프로그램 코드로 들어가서 실행
- 만든 파일(File)이 실행할 수 있는, 이식 가능한 다른 곳에 옮겨져도(Portable) 실행이 가능하도록(Executable) 만들어 놓은 포맷(Format)

<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/0139f682-7bf8-4ed3-b38c-0219e66fa311)

PE 파일 생성과정은 위와 같다.
1. 소스코드 작성
2. 바이너리로 변경(컴파일)
3. 필요한 라이브러리 연결(링킹)
4. exe파일로 빌드(빌드)


PE 파일 생성 과정도 중요하지만, PE 헤더도 중요하다. PE 파일을 빌드할 때, 파일 실행 시 필요한 정보들을 약속된 규약에 맞춰 PE 파일을 헤더에 기입한다.<br>
그렇기 때문에 PE 파일의 헤더 정보를 보고 운영체제가 해당 프로그램을 실행시키게 된다.<br>

그래서 위키피디아에서는 "PE 포맷은 윈도우 로더가 실행 가능한 코드를 관리하는데 필요한 정보를 캡슐화한 데이터 구조체" 라고 설명 했다.

<br>

```
typedef struct _IMAGE_DOS_HEADER // DOS .EXE header
{
   WORD e_magic; // Magic number
   WORD e_cblp; // Bytes on last page of file
   WORD e_cp; // Pages in file
   WORD e_crlc; // Relocations
   WORD e_cparhdr; // Size of header in paragraphs
   WORD e_minalloc; // Minimum extra paragraphs needed
   WORD e_maxalloc; // Maximum extra paragraphs needed
   WORD e_ss; // Initial (relative) SS value
   WORD e_sp; // Initial SP value
   WORD e_csum; // Checksum
   WORD e_ip; // Initial IP value
   WORD e_cs; // Initial (relative) CS value
   WORD e_lfalc; // File address of relocation table
   WORD e_ovno; // Overlay number
   WORD e_oemid; // OEM identifier (for e_oeminfo)
   WORD e_oeminfo; // OEM information; e_oemid specific
   WORd e_res2[10]; // Reserved words
   LONG e_lfanew; // File address of new exe header
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

<br>

PE 파일은 IMAGE_DOS_HEADER 구조체로 시작한다. 이는 DOS 시절에 사용되던 실행 파일의 헤더로서 실행 파일이 DOS에서 실행 되었을 때 DOS 운영체제에 알려줘야 할 정보들이 담겨 있다.<br>
이것은 DOS 운영체제를 위해 제공하는 정보들이기 때문에 현재 Windows 환경에서는 필요치 않다. 대다수 필드들은 무시되며 e_magic 및 e_lfanew 필드만이 유용한 정보가 된다.<br>
e_magic 필드는 IMAGE_DOS_HEADER의 시작을 나타내는 것으로 항상 "MZ"라는 문자열로 시작한다. 그리고 e_lfanew 필드는 IMAGE_DOS_HEADER 다음에 나오는 헤더 파일의 오프셋 값을 가지고 있다.<br>
그 외 필드들은 Windows에서 파일을 실행 시켰을 때 이용되지 않는다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/79e2c7fa-d3ad-47a4-9dbf-4ea65b6cef92)

위의 값들은 DOS Mode 실행 시 Dos Stub 실행을 목적으로 지정된 값들로 이 헤더 뒤에 따르는 문자열 "This program cannot be run in DOS mode"을 출력해 주기 위해 지정된 값들이다.<br>
Dos가 아닌 Windows 모드에서 실행 시켰을 경우에는 이용되지 않는다. 그러나 몇몇 악성코드를 분석하다 보면 실제 사용되지 않는 필드 값들이 악성코드에 의해 사용될 수 있다.<br>
악성코드에서 많이 사용되는 실행압축 방식의 하나인 Upack의 IMAGE_DOS_HEADER 부분은 아래에서 살펴본다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/fde65a22-f79c-4b1c-acce-4b8a9bc3b331)

위의 사진은 악성 코드에서 사용된 구조인데 IMAGE_DOS_HEADER 와는 확연한 차이가 보인다. e_magic 및 e_lfanew 필드 이외의 값들이 "Kernel32.dll","LoadlibraryA","GetprocAddrss" 등의 문자열로 채워진 것을 확인 할 수 있다.<br>
Upack에서 필요한 라이브러리 함수(DLL)들을 가져다 쓰는 루틴을 위해 존재한다.

<strong>학습한 블로그는 2008년에 작성된것으로 현재 악성코드분석하면서 IMAGE_DOS_HEADER부분에 위의 사진 처럼된 악성코드는 본적은 없다.</strong>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/0284f307-3ccd-452e-9cdf-422bffbb1ae0)

IMAGE_DOS_HEADER의 e_lfanew 필드는 다음에 올 헤더 위치의 파일 오프셋을 가리키고 있다. 즉, 실제 PE 파일의 시작이라고 할 수 있는 IMAGE_NT_HEADER의 시작 오프셋 값을 가지고 있다<br>
아래는 IMAGE_NT_HEADER의 구조체 이다.<br>

```
typedef struct _IMAGE_NT_HEADERS
{
   DWORD Signature;
   IMAGE_FILE_HEADER FileHeader;
   IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADER32, *PIMAGE_NT_HEADER32;
```

<br>

NT_HEADERS
- Signature
- File Header
- Optional Header

<br>
<strong>Signature</strong><br>
첫 필드인 SIgnature는 PE 헤더의 시작을 알리는 값으로 "PE"라는 문자열이 들어가게 된다. 그렇기 때문에 IMAGE_DOS_HEADER의 e_lfanew 필드가 가리키는 위치를 가보면 "PE" 같은 문자열이 나난다.<br>
IMAGE_FILE_HEADER 와 IMAGE_OPTIONAL_HEADER32는 PE 구조체의 세부적인 값들로 지정된 구조체이다. 그럼 e_flanew 필드는 언제나 자신보다 뒤에 있는 영역을 가리키고 있는 것일까?<br>
일반적인 실행 파일의 경우는 그렇다고 할 수 있지만, 악성코드들은 다른 모습이 보이기도 한다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/951e1eac-7edf-4dde-a395-898d03fbf030)

일반적인 실행파일과 다르게 IMAGE_DOS_HEADER의 e_lfanew 필드가 자신보다 앞의 영역을 가리키고 있다.<br>
이와 같이 다소 일반적인 않은 형태로 구조체가 정의 되더라도 각 필드에 맞는 값들이 채워진다면 운영체제가 파일을 읽어 들여서 실행하는 것에는 문제가 되지 않는다.<br>
특히 IMAGE_DOS_HEADER의 첫필드와 마지막 필드를 제외하고는 현재 이용되지 않기 때문에 전혀 문제가 되지 않는다. 단 IMAGE_NT_HEADER 구조체의 시작이 앞으로 이동하게 되면 e_lfanew 필드 값이 PE 헤더 값과 겹치게 되므로, e_lfanew 필드 값과 매칭되는 PE 헤더의 값(baseOfData)에 문제가 없어야 한다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/b814424b-24d9-441f-aeef-15f7a44d2f1a)

IMAGE_DOS_HEADER의 e_lfanew 필드는 파일 오프셋으로 0x00000010을 가리키고 있다. 그리고 IMAGE_NT_HEADER의 앞부분에 위치한 이용되지 않는 IMAGE_DOS_HEADER의 영역에 "KERNEL32.dll" 문자열을 숨겨 두었다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/2a049d57-c421-4cc0-a1f5-311cb7d6f222)

<br>

<strong>File Header 주요 맴버</strong><br>
1. Machine - CPU 별 고유한 값(Intel x86 = 0x014c, intel x64 = 0x0200 값)
2. NumberOfSections - 섹션의 개수 
3. SizeOfOptionalHeader - Optional Header의 크기를 나타냄
4. Characteristics - 파일의 속성을 나타냄 

<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/7e300c03-62c3-41bd-b070-f89b8d17e431)

<strong>Optional Header 주요 맴버</strong><br>

1. Magic - Magic 넘버는 32비트면 0x10B, 64비트면 0x20B
2. AddressOfEntryPoint - EP의 RVA 값을 가지고 있음, 프로세스의 시작 위치 주소값(RVA)
3. ImageBase - PE 파일이 로딩되는 시작 주소 , 파일을 메모리에 로딩 후 EIP레지스터의 값 = ImageBase + AddressOfEntryPoint
4. SectionAlignment - 메모리에서 섹션의 최소 단위
5. FileAlignment - 파일에서 섹션의 최소 단위
6. SizeOfImage - PE 파일이 메모리에 로딩되었을 때의 크기
7. SizeOfHeader - PE 헤더의 전체 크기
8. Subsystem - GUI,CUI 버전인지 드라이브파일인지 나타냄(1=드라이버, 2=GUI, 3=CUI)
9. NumberOfRvaAndSizes - DataDirectory 배열의 개수를 나타냄
10. DataDirectory - IMAGE_DATA_DIRECTORY 구조체의 배열, 각 항목은 정해진 값을 가지고 각 항목 table이 어디에 위치해 있는지 RVA 값과 크기를 나타냄

<br>

# More Imformation

PE 헤더 정보를 메모리에 매핑
1. 실제 프로세스를 위한 메모리 할당
2. 섹션 정보를 메모리에 복사
3. import 정보 처리
4. 기준 재배치 처리

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/645d177f-8c12-409c-8d95-5277a9086e60)

- 프로그램의 실행 영역(실제 데이터 영역과 관련된 헤더 정보) : IMAGE_SECTION_HEADER.text
- 실제 데이터 영역 : SECTION .text

즉 헤더부분은 실제 데이터 영역의 관련된 정보들이 있고 실제 데이터 값들은 바디 섹션에 들어간다.

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/71502809-da21-4a04-a2f2-817c86b135ca)

<strong>특징</strong>
1. 헤더 정보까지는 File 과 Memory 동일하다.
2. SECTION .text = 실행 영역
3. offset은 상대 주소 개념이고 address는 (메모리 상의) 실제 주소이다.
4. SECTION 부분에서 Memory의 Padding 부분이 더 길어짐(실행할 때 메모리 특정 영역 할당 또는 점프해서 돌아오는 등의 행동을 대비해 더 길어짐)
5. File 은 메모리가 차곡차곡있는 것이 좋음
6. Memory 는 주소 매핑을 편리하게 하기 위해 기준이 되는 값을 정하고 그 기준을 바탕으로 새로운 섹션에 값을 넣는다.

<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/30436b07-50bd-4aa9-88bc-aeb1bb231199)

일반적으로 같은 플랫폼에서 같은 실행 파일의 경우 직접 링커를 통해 실행 파일을 만들지 않는 이상 같은 속성을 가지게 된다. 그러나 실행 파일 헤더의 특정 몇몇 부분이 다를 경우에는 실행 파일이 감염 되었음을 의심해 볼 수 있다.<br>
감염되기 이전의 IMAGE_NT_HEADER에 속한 IMAGE_FILE_HEADER의 값과 감염된 이후 박스안의 NumberOfSections 필드를 살펴보면 감염된 이후에 값이 1만큼 증가한 것을 확인 할 수 있다.<br>
원본 파일의 맨 뒤에 바이러스를 추가 시키는 Appending 바이러스의 한 유형으로 바이러스를 추가할 부분을 만들기 위해 섹션의 수를 하나 증가시켜 놓았다.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/c4d9e6c0-8cee-4c48-b653-e05a62ed9fc8)

섹션의 이름은 .adate임을 알 수 있다. SizeOfRawData는 파일상에서 섹션의 크기를 대해 알 수 있으며, 치료할 때 사용된다. PointerToRawData는 파일상의 섹션 위치를 알 수 있으며, 이것 또한 치료할 때 사용된다.<br>
Characteristics은 해당 섹션의 속성을 나타내는 플래그의 집합이다.
- IMAGE_SCN_CNT_CODE
- EXECUTE
- READ
- WRITE


# VA , RVA , RAW

VA : 프로세스 가상 메모리의 절대주소(실제 메모리에 로딩되는 주소)
RVA : ImageBase로 부터의 상대 주소(메모리에 로딩된 상태)
1. PE가 메모리에 적재되기 전에 기본 ImageBase는 0
2. PE안에서는 상대주소로 주소가 적힘 
  - 메모리에 적재시 절대주소가 들어가게 된다면 Relocation이 어렵기 때문
  - 상대주소가 들어가면 ImageBase에서 얼마만큼 떨어진 곳으로 이동하면 되기때문에 Relocation이 쉽게되어 상대주소를 사용

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/d4fbe698-5080-4753-80e8-9fa2324235ea)

파일이 실행되기 전에 주소를 File Offset이라고 하며, RAW라고 부른다.

실행 되기 전의 주소이기 때문에 파일 자체를 분석해야 할 때 주로 이 주소를 기반으로 데이터를 확인한다.<br>

RVA, VA를 가지고 RAW를 계산할 수 있으며 계산 방법은 다음과 같다<br>
1. 섹션에서 메모리의 주소(RVA)가 속해져 있는 섹션을 찾는다
2. 계산식으로 계산한다 RAW = RVA - VA + PointerToRawData


# IAT

IAT(Import Address Table)에는 Windows 운영체제의 핵심 개념인 Process, Memory, DLL 구조 등에 대한 내용이 함축되어 있다.<br>
IAT란 프로그램이 어떤 라이브러리에서 어떤 함수를 사용하고 있는지를 기술하는 테이블이다.<br>

DLL(Dynamic Linked Library)는 프로그램에 라이브러리를 포함시키지 않고 별도의 파일(DLL)로 구성하여 필요할 때마다 불러 쓰는 방식으로,<br>
한 번 로딩된 DLL은 다른 Process들이 공유해서 사용함으로써 memory를 효율적으로 사용할 수 있는 방법이다.<br>

# EAT

EAT(Export Adderss Table)은 라이브러리 파일에서 제공하는 함수를 다른 프로그램에서 가져다 사용할 수 있도록 하는 메커니즘이다.<br>
EAT를 통해서만 해당 라이브러리에서 export 하는 함수의 시작 주소를 정확히 구할 수 있다.<br>

```
typedef struct _IMAGE_EXPORT_DIRECTORY { 
	DWORD Characteristics; 
	DWORD TimeDateStamp ; // creation time date stamp 
	WORD MajorVersion; 
	WORD MinorVersion; 
	DWORD Name; // address of librar y file name 
	DWORD Base; // ordinal base 
	DWORD NumberOfFunctions; // number of functions 
	DWORD NumberOfNames; // number of names 
	DWORD AddressOfFunctions ; // address of function start address array

	DWORD AddressOfNames; // address of function name stri ng array 
	DWORD AddressOfNameOrdinals; // add ress of ordinal array 
} lMAGE_EXPORT_DIRECTORY, *PlMAGE_EXPORT_DIRECTORY;
```

<br>

EAT는 IMAGE_EXPORT_DIRECTORY구조체 안에 기록 되어 있는데, 이것 또한 IAT처럼 PE바디 안에 있고, 이것은 PE구조에 단 한개만 존재한다.<br>
IMAGE_OPTIONAL_HEADER.DataDirectory[1].VirtualAddress 값이 IAT의 구조체 배열 시작 주소였다면,<br>
IMAGE_OPTIONAL_HEADER.DataDirectory[0].VirtualAddress 값이 실제 IMAGE_EXPORT_DIRECTORY 구조체 배열의 시작 주소이다. 이것도 물론 RVA로 된 값이다.<br>



# 요약

Q. PE 파일에 대해 설명하시오<br>
A. <br>
PE 파일은 만든 파일이 실행 할 수 있는 다른 곳에 옮겨져도 실행이 가능하도록 만들어 놓은 포맷입니다. <br>
PE 파일 생성과정은 소스코드 작성 -> 바이너리로 변경 -> 필요한 라이브러리 연결 -> exe 및 dll 으로 알맞게 파일을 빌드합니다. <br>
PE 파일을 빌드 할 때, 파일 실행 시 필요한 정보들을 규약에 맞춰 PE 헤더 기입 하기 때문에 PE 파일 헤더 정보를 보고 운영체제가 해당 프로그램을 실행시키게 됩니다. <br>
PE 파일은 IMAGE_DOS_HEADER 구조체로 시작하며 e_magic 필드에는 "MZ" 라는 문자열로 시작하고 대다수 필드는 사용이 되지 않는다.<br>
그리고 NT_HEADER 구조체가 나오는데 Signature File Header Optional Header가 있다.<br>
긱 주요 맴버들로는 Machine, NumberOFSections, ImageBase등이 있다.<br>

VA 가상 메모리 절대 주소이고, RVA에는 ImageBase부터 상대 주소이다.<br>
PE안에서는 상대주소로 적힌다. 그 이유는 메모리에 적재시 절대주소가 들어가면 재배치가 어렵기 때문이다. 그래서 상대주소를 사용해서 ImageBase에서 얼마만큼 떨어진 곳에 이동하면 되기 때문에 재배치가 수월하다.<br>

VA 주소 확인하는 방법은 RVA + Imagebase 이다.<br>

RAW는 PE파일이 로딩되기 전에 Fileoffset이다.<br>
실행되기 전의 주소이기 때문에 파일 자체를 분석해야 할 때 주로 이 주소를 기반으로 데이터 확인한다. <br>

# 출저 
https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=wsi5555&logNo=221259915738<br>
https://mocharoll.tistory.com/16<br>
https://hxxyxxn-1238.tistory.com/65<br>
https://mocharoll.tistory.com/15<br>
https://skensita.tistory.com/entry/%EC%95%85%EC%84%B1%EC%BD%94%EB%93%9C-%EB%B6%84%EC%84%9D%EA%B0%80-%EC%9E%85%EC%9E%A5%EC%97%90%EC%84%9C-%EB%B3%B8-PE-%EA%B5%AC%EC%A1%B0<br.
