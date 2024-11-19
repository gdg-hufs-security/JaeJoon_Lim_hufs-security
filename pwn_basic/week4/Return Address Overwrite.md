# Retrun Address Overwrite 실습해보기


# 예제
```
// Name: rao.c
// Compile: gcc -o rao rao.c -fno-stack-protector -no-pie

#include <stdio.h>
#include <unistd.h>

void init() {
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
}

void get_shell() {
  char *cmd = "/bin/sh";
  char *args[] = {cmd, NULL};

  execve(cmd, args, NULL);
}

int main() {
  char buf[0x28];

  init();

  printf("Input: ");
  scanf("%s", buf);

  return 0;
}
```
  1) 문제점
     - scanf("%s", buf)에서 %s는 입력의 길이를 제한하지 않기 때문에 버퍼보다 큰 데이터를 입력하면 오버플로우가 발생할 수 있다.
       - sol) "%[n]s" 사용

     - 이외에 코드를 짤 때 버퍼의 크기를 같이 입력하는 다음 함수들을 지향한다.
       - strcpy  -> strncpy
       - strcat -> strncat
       - sprintf -> snprintf
       - fgets, memcpy

     - 이 예제에서는 크기가 0x28인 버퍼에 그 이상의 데이터를 주면 버퍼 오버플로우를 발생시켜 main함수의 반환 주소를 덮을 수 있다.
    

  2) 코어 파일??


  3) 스택 프레임 구조 파악
     - 스택 버퍼에 오버플로우를 발생시켜 반환주소를 덮으려면, 우선 해당 버퍼가 스택 프레임의 어디에 위치하는지 조사해야 한다.


  4) main의 어셈블리 코드
     ```
     pwndbg> nearpc
       0x400706             call   printf@plt 
    
       0x40070b             lea    rax, [rbp - 0x30]
       0x40070f             mov    rsi, rax
       0x400712             lea    rdi, [rip + 0xab]
       0x400719             mov    eax, 0
     ► 0x40071e             call   __isoc99_scanf@plt <__isoc99_scanf @plt>
            format: 0x4007c4 ◂— 0x3b031b0100007325 /* '%s' */
            vararg: 0x7fffffffe2e0 ◂— 0x0
     ```
     - scanf가 입력을 저장할 목적지는 rbp-0x30이다. 즉, 오버플로우를 발생시킬 버퍼는 rbp-0x30에 위치한다.
     - 스택 프레임을 그려보면 다음과 같다.
       | buf             |  <- rbp-0x30
       |                 |
       | SFP             |  <- rbp
       | return address  |  <- rbp+0x8
       - 버퍼의 크기가 0x30인 것을 알 수 있다. 우리는 0x38크기만큼 데이터를 입력하면 반환 주소를 조작할 수 있다.
       - Q) SFP는 무엇일까?
      

  5) 셸을 실행해주는 get_shell 함수의 주소 찾기
     ```
      void get_shell() {
        char *cmd = "/bin/sh";
        char *args[] = {cmd, NULL};
      
        execve(cmd, args, NULL);
      }
     ```
     - get_shell() 함수를 보면 셸을 실행해주는 코드를 볼 수 있다. 우리는 이 함수의 주소로 main함수의 반환 주소를 덮어서 셸을 획득할 수 있다.
     - 주소 찾는 방법
       ```
        $ gdb rao -q
        pwndbg> print get_shell
        $1 = {<text variable, no debug info>} 0x4006aa <get_shell>
        pwndbg> quit
       ```
       - 가상 환경에서 pwndbg로 들어가 get_shell 함수의 주소를 알 수 있다.  0x4086aa 임을 알 수 있다.
      
  6) 페이로드 구성
     - 이제 익스플로잇에 사용할 페이로드를 구성해야 한다.
     - 페이로드란 무엇일까?) => 시스템 해킹에서 페이로드는 **공격을 위해 프로그램에 전달하는 데이터**를 의미한다.
     - ???

  7) 페이로드 작성? -> 적절한 **엔디언**을 적용해서 프로그램에 전달해야 한다.
       - 엔디언이란 메모리에서 데이터가 정렬되는 방식이며, 주로 리틀 엔디언과 빅 엔디언이 사용된다.
       - 리틀 엔디언에서는 MSB(가장 왼쪽 바이트)가 가장 높은 주소에 저장되고, 빅 엔디언에서는 데이터의 MSB가 가장 낮은 주소에 저장된다.
       - 쉽게 말해서, 0x123456이 있다면 빅 엔디언은 1바이트(두 자리)를 그대로 주소에 저장하는 방식이고, 리틀 엔디언은 거꾸로 저장한다고 보면 된다.
       - 예시에서 get_shell주소는 0x4086aa이다. 이를 리틀 엔디언으로 바꾸면 "\xaa\x86\x40\x00\x00\x00\x00\x00" 이다.
         - 왜 리틀 엔디언으로 바꿀까?) => x86-64 아키텍처가 리틀 엔디언 방식을 기본으로 채택했기 때문이다.
         - get_shell의 주소는 0x4086aa로 3바이트이지만 리틀 엔디언으로 바꾸면 왜 8바이트로 되는 걸까?) => x86-64는 64비트 레지스터와 64비트 주소 공간을 사용하는 아키텍처이다. 따라서 모든 주소는 기본적으로 8바이트(64비트)로 표현된다.

  8) 이후 풀이는 나중에...



# C언어에서 자주 사용되는 문자열 입력 함수와 패턴들
  1) scanf("%s", buf)
     - 예제에서 보았던 함수이다. 입력받는 길이에 제한이 없으며 버퍼의 널 종결을 보장하지 않음.
  2) gets(buf)
     - scanf()와 동일하다. 입력받는 길이에 제한이 없으며 버퍼의 널 종결을 보장하지 않는다. 입력의 끝에 널바이트를 삽입하므로, 버퍼를 꽉채우면 널바이트로 종결되지 않는다. -> 이후 문자열 관련 함수를 사용할 때 버그가 발생하기 쉬움.
  3) scanf("%[n]s", buf)
     - n만큼만 입력받는다. 단, n을 설정할 때 버퍼의 크기보다 1작게 설정해야 한다. 널바이트를 고려해야 하기 떄문.
     - gets와 마찬가지로 버퍼의 널 종결을 보장하지 않는다. => 즉, 입력 데이터를 처리할 때 입력의 끝에 자동으로 널 바이트를 추가해 문자열을 종결시키지 않는다는 의미이다.
  4) fgets(buf, len, stream)
     - 특징은 **버퍼의 널 종결을 보장**한다는 점이다. 단, 데이터를 모두 담고 싶다면 len을 버퍼의 크기보다 작게 설정해야 한다. 만약 버퍼의 크기가 30이고 len 또한 30이라면, 29바이트만 저장되고, 마지막 바이트는 유실되어 널바이트로 채워진다.


