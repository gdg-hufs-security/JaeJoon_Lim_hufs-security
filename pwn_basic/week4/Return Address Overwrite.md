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
