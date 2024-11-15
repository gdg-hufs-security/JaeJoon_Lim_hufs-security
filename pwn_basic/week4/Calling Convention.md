#  함수 호출 규약
  - **함수 호출 및 반환에 대한 약속**이다.

  - 호출자의 상태 및 반환 주소
    - 한 함수에서 다른 함수를 호출할 때 프로그램 실행 흐름은 다른 함수로 이동한다. 그리고 호출한 함수가 반환하면, 다시 원래의 함수로 돌아와서 기존 실행 흐름을 이어나간다. 이때 필요한 것들이 **호출자의 상태 및 반환 주소**이다.
   
  - 호출자와 피호출자의 역할
    - 호출자 : 피호출자가 요구하는 인자를 전달해줘야 함.
    - 피호출자 : 반환 값을 전달해줘야 함.

  - 누가 함수 호출 규약을 적용하는가?
    - **컴파일러**
      - 프로그래머가 고수준 언어로 코드를 작성하면, 컴파일러가 호출 규약에 맞게 코드를 컴파일한다.
        - 컴파일러가 아키텍처에 맞게 호출 규약을 적용해주니 사용자가 딱히 몰라도 된다.
        - 컴파일러는 지원하는 호출 규약 중, CPU아키텍처에 적합한 것을 선택한다.
        - CPU의 아키텍처가 같아도 컴파일러에 따라 다른 호출 규약을 적용할 수도 있다.
          
  - 우리가 함수 호출 규약을 배워야하는 이유
    - 컴파일러 도움없이 어셈블리 코드를 작성해야 하는 경우
    - 어셈블리로 작성된 코드를 읽고자 하는 경우
    - 이 두 경우에 함수 호출 규약을 알아야 한다.
   


# 함수 호출 규약의 종류
  - x86
    - **cdecl**
    - stdcall
    - fastcall
    - thiscall

  - x86-64
    - **System V AMD64 ABI의 Calling Convention**
    - MS ABI의 Calling Convention
   


#x86-64 호출 규약: SYSV
  - 리눅스는 SYSTEM V(SYSV) Application Binary Interface(ABI)를 기반으로 만들어졌다.
    - 리눅스에서 실행되는 수많은 프로그램들은 서로 다른 개발자들에 의해 만들어진다. 프로그램마다 파일 형식, 함수 호출 방법, 데이터를 주고 받는 규칙이 제각각이면 운영체제는 이 프로그램들을 제대로 실행할 수 없다. 그래서 SYSV라는 표준 규칙을 만들어 모든 프로그램이 이 규칙을 따르도록 약속한 것이다.
   
  - 그렇다면 어떤 규칙들이 있을까?
    - 파일 형식 (ELF)
      - 모든 실행 파일은 ELF 형식으로 저장된다. 운영체제는 이 형식을 알고 있으니 파일을 읽고 실행할 수 있다.
    - 함수 호출 규칙
      - 프로그램이 함수를 호출할 때 어떤 값이 레지스터에 들어가는지 정해져 있다. 이 부분은 예전에 셀 코드를 작성할 때 경험해봤다.
      - 예를 들어, 첫 번째 인자는 RDI 레지스터, 두 번째 인자는 RSI 레지스터에 넣는 식이다.
    - 링킹
      - 링킹이란?) 여러 개의 코드 조각을 하나의 **실행 파일**로 합치는 과정이다.
      - (이 부분은 공부를 해봐야겠다.)

  - 우분투에서  file /bin/ls 명령어를 입력했을 때, SYSV문자열이 포함된 것을 볼 수 있다.
    ```
    root@노트북:~# file /bin/ls
    /bin/ls: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=36b86f957a1be53733633d184c3a3354f3fc7b12, for GNU/Linux 3.2.0, stripped
    ```

  - SYSV에서 정의한 함수 호출 규약의 특징
    - 6개의 인자를 RDI, RSI, RDX, RCX, R8, R9에 순서대로 저장하여 전달합니다. 더 많은 인자를 사용해야 할 때는 스택을 추가로 이용합니다.
    - Caller에서 인자 전달에 사용된 스택을 정리합니다.
      - 함수 호출 시 7번째 인자부터 스택에 정리된다. 스택에 저장된 인자는 함수가 끝난 후에도 메모리에 남아 있습니다. 만약 Caller가 이를 정리하지 않으면 메모리 누수, 이후 함수 호출 시 스택이 잘못된 위치에 있을 수 있다.
    - 함수의 반환 값은 RAX로 전달합니다.



# x86호출 규약: cdecl
  - x86아키텍처는 레지스터의 수가 적으므로, **스택**을 통해서 인자를 전달한다.
  -  인자를 전달하기 위해 사용한 스택을 호출자가 정리하는 특징이 있습니다. 스택을 통해 인자를 전달할 때는, 마지막 인자부터 첫 번째 인자까지 거꾸로 스택에 push한다.



### Quiz
Q). 아래 코드를 컴파일 했을 때, 컴파일된 어셈블리 코드 중 빈칸에 들어갈 것으로 가장 적절할 것을 고르시오.
```
// Name: callconv_quiz.c
// Compile: gcc -o callconv_quiz callconv_quiz.c -m32
int __attribute__((cdecl)) sum(int a1, int a2, int a3){
	return a1 + a2 + a3;
}
void main(){
	int total = 0;
	total = sum(1, 2, 3);
}
```
  1) C언어 파일로 만들고 gcc -m32 -o callconv_quiz callconv_quiz.c   명령어로 컴파일.
  2) gdb -q callconv_quiz 명령어로 파일을 연다.
  3) b *main 으로 우리가 보고 싶은 main함수 시작 주소로 breakpoint 생성
  4) r 명령어로 main함수 보기
  5) 문제점 : 
     ```
      main:   
     0x080483ed <+0>:	push   ebp
     0x080483ee <+1>:	mov    ebp,esp
     0x080483f0 <+3>:	sub    esp,0x10
     0x080483f3 <+6>:	mov    DWORD PTR [ebp-0x4],0x0
     0x080483fa <+13>:	(a)
     0x080483fc <+15>:	(b)
     0x080483fe <+17>:	(c)
     0x08048400 <+19>:	call   0x80483db <sum>
     0x08048405 <+24>:	(d)
     0x08048408 <+27>:	mov    DWORD PTR [ebp-0x4],eax
     0x0804840b <+30>:	nop
     0x0804840c <+31>:	leave  
     0x0804840d <+32>:	ret 
     ```
     문제에서 주어진 어셈블리 코드와 내 터미널의 어셈블리 코드 구성이 달랐음

  6) 해결법 : 내가 컴파일한 코드에는 **최적화**가 되어 있었음. 또한, 정적/동적 부분에서도 차이점이 있다고 했음.
  7) 새로운 코드로 컴파일하기  :  gcc -m32 -O0 -fno-pie -no-pie -o callconv_quiz callconv_quiz.c
  8) 추출한 어셈블리 코드
     ```
     ► 0x555555555149 <main>       endbr64
       0x55555555514d <main+4>     push   rbp
       0x55555555514e <main+5>     mov    rbp, rsp                   RBP => 0x7fffffffe140 ◂— 1
       0x555555555151 <main+8>     sub    rsp, 0x10                  RSP => 0x7fffffffe130 (0x7fffffffe140 - 0x10)
       0x555555555155 <main+12>    mov    dword ptr [rbp - 4], 0     [0x7fffffffe13c] <= 0
       0x55555555515c <main+19>    mov    edx, 3                     EDX => 3
       0x555555555161 <main+24>    mov    esi, 2                     ESI => 2
       0x555555555166 <main+29>    mov    edi, 1                     EDI => 1
       0x55555555516b <main+34>    call   sum                         <sum>
    
       0x555555555170 <main+39>    mov    dword ptr [rbp - 4], eax
       0x555555555173 <main+42>    nop
     ```
