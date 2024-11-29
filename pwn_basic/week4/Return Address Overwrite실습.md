# 문제 파일
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

  1) 문제 파악
     - main함수를 보면 0x28크기의 버퍼가 있다. %s는 문자열 길이를 제한해서 받지 않기 때문에 버퍼 오버플로우가 발생할 수 있다.
     - 40바이트 이상의 데이터를 넣어서 main함수의 반환 주소를 셸을 실행하는 코드가 있는 get_shell함수로 변경해야 한다.
    
  2) 컴파일 하기
     - 우선 이 코드를 컴파일해서 get_shell()함수 저장된 메모리 주소를 알아야 한다.

  3) gdb로 get_shell가 저장된 메모리 주소를 파악하기
     - $ gdb ./rao    // 가상 환경에서 rao파일을 로드한다.
     - $ p &get_shell  // get_shell의 시작 주소를 확인한다. 또는  $ info address get_shell 명령어를 사용해도 좋다.
     -  ~~0x4011dd  라는 주소가 나왔다.~~
      
    
      
    
  4) 리틀 엔디언 형식으로 바꾸기
     - 64비트 아키텍처에서는 주소는 8바이트로 표현된다. 때문에 리틀 엔디언 형식으로 변경하면 \xdd\x11\x40\x00\x00\x00\x00\x00 와 같다.
    
  5) 익스플로잇 하기
     - 
