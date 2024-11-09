# 셸
  - 셸이란?) 사용자와 운영체제 커널 간의 인터페이스이다. 사용자가 명령어를 입력하면 셸은 이를 해석하고 커널에 전달해 실행한다. 
  - 셸 획득 == 시스템 해킹의 성공 이라고 볼 수 있다.
  - 리눅스에서 여러 가지 셸 프로그램 종류
    > sh : 가장 기본적인 셸 프로그램
    
    > bash : sh에서 발전한 셸로, 가장 많이 사용된다.
    
    > zsh
    
    > tsh

# execve 셸코드
  - 뭔가?) 임의의 프로그램을 실행하는 셸코드이다. -> 이를 이용하면 서버의 셸을 획득할 수 있다.


# (아직 감이 잘 안 온다. 실제 코드를 살펴보자.)


# execve("/bin/sh", null, null)
  - execve 셸코드 특징으로는 **execve 시스템 콜로만 구성된다**는 것이다.

```
syscall  rax     arg0 (rdi)              arg1 (rsi)                  arg2 (rdx)
execve   0x3b    const char *filename    const char *const *argv     const char *const *envp
```
  1) 여기서 argv는 실행파일에 넘겨줄 인자, envp는 환경변수이다.
     - argv란 무엇일까?)
       > argument vector의 줄임말로, 프로그램에 넘겨줄 인자들을 저장하는 **배열**이다.
       > 즉, 프로그램이 실행될 때 사용할 수 있는 추가 정보들을 전달하는 역할을 한다.
         ```
         ls -1 /home/user
         ```
         라는 명령어가 있을때, argv[0] = "ls"   argv[1] = "-1"  argv[2] = "/home/user" 로 이해할 수 있다.
       > 여기서 argv[0] 에는 항상 프로그램 이름이 들어간다. 예시에서는 sh가 들어감.

      - 환경변수란?   // 구체적인 건 나중에 알아보자.
        > 환경 변수는 컴퓨터에서 프로그램이 실행될 때 필요한 설정 정보나 값을 저장하는 특별한 변수이다.

  2) 우리는 sh만 실행하면 되므로 다른 값들은 전부 null로 설정해줘도 됩니다.
     - execve 셸코드에서는 sh 말고는 추가 인자가 필요 없다. -> 그래서 argv = ["sh", NULL] 로 설정한다.

  3) rax 값
     - execve 시스템 호출 번호는 0x3b이다.(59) 즉, rax에는 0x3b값이 저장된다.

  4) rdi 값
     - execve()의 첫 번째 인자는 filename, 즉 **실행할 프로그램의 경로**가 들어간다.
     - 우리는 /bin/sh를 실행할 것이다. 그래서 rdi에는 **/bin.sh의 주소**가 들어간다.
     - /bin/sh 문자열이 메모리에 0x68732f6e69622f값으로 저장되어 스택에 들어간다. 스택의 주소가 rdi에 들어간다.
    
  5) rsi 값
     - execve()에서 두 번째 인자는 argv, 즉 프로그램에 넘겨줄 인자 목록이다.
     - 이번 예시에는 인자가 필요없으므로, NULL로 설정한다.

  6) rdx 값
     - ececve()에서 세 번쨰 인자는 envp, 즉 환경 변수 목록이다.
     - 이번 예시에서 환경 변수가 필요 없으므로, NULL로 설정한다.

# 어셈블리로 작성한 execve셸코드
```
;Name: execve.S

mov rax, 0x68732f6e69622f
push rax
mov rdi, rsp  ; rdi = "/bin/sh\x00"
xor rsi, rsi  ; rsi = NULL
xor rdx, rdx  ; rdx = NULL
mov rax, 0x3b ; rax = sys_execve
syscall       ; execve("/bin/sh", null, null)
```

# evecve 셸코드 컴파일 및 실행
  1) 스켈레톤 코드 + execve셸코드
```
// File name: execve.c
// Compile Option: gcc -o execve execve.c -masm=intel

__asm__(
    ".global run_sh\n"
    "run_sh:\n"

    "mov rax, 0x68732f6e69622f\n"
    "push rax\n"
    "mov rdi, rsp  # rdi = '/bin/sh'\n"
    "xor rsi, rsi  # rsi = NULL\n"
    "xor rdx, rdx  # rdx = NULL\n"
    "mov rax, 0x3b # rax = sys_execve\n"
    "syscall       # execve('/bin/sh', null, null)\n"

    "xor rdi, rdi   # rdi = 0\n"
    "mov rax, 0x3c	# rax = sys_exit\n"
    "syscall        # exit(0)");

void run_sh();

int main() { run_sh(); }
```

  2) C언어로 파일을 만들고 우분투에서 명령어를 입력해 해당 파일의 디렉토리로 이동
```
cd /mnt/c/dreamhackpractice/
```

  3) bash 셸 환경으로 들어가기
```
bash
```

  4) gcc컴파일러로 execve.c 파일 만들기
```
bash$ gcc -o execve execve.c -masm=intel
```

  5) execve 셸코드를 실행
```
bash$ ./execve
```
   - 셸코드를 실행하면 사용자는 대화형 셸(sh) 환경에 들어간다.
   - 우분투에 #이 생기면서 #프롬프트로 진입한다.

  6) 사용자의 정보 출력하기
```
id
```
  - 출력값 : uid=1000(dreamhack) gid=1000(dreamhack) groups=100
  - 이런 양식으로 출력된다.
