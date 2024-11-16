# 스택 버퍼 오버플로우
  1) 간단하게 설명하자면 스택 영역에 할당된 메모리 범위를 초과하여 데이터를 덮어쓰는 취약점이다.


# 스택 버퍼 오버플로우 vs 스택 오버플로우
  1) 스택 오버플로우
     - 스택 영역은 실행중에 크기가 동적으로 확장될 수 있는데, 스택 오버플로우는 스택 영역이 너무 많이 확장돼서 발생하는 버그를 뜻한다.

  2) 스택 버퍼 오버플로우
     - 스택에 위치한 버퍼에 버퍼의 크기보다 많은 데이터가 입력되어 발생하는 버그를 뜻한다.


# 버퍼
  1) 스택 버퍼 오버플로우는 스택의 버퍼에서 발생하는 오버플로우를 뜻한다. 그렇다면 버퍼는 무엇일까?
    - 버퍼는 **데이터가 목적지로 이동하기 전에 보관되는 임시 저장소**이다.

  2) 탄생 배경
     - 데이터의 처리속도가 서로 다른 두 장치 사이를 데이터가 오고 갈때, 데이터 유실이 있을 수 있다. 이를 대비해 버퍼라는 저장 장치를 사용한다. 송신 측은 버퍼로 데이터를 전송하고 수신 측은 버퍼 속 데이터를 꺼내 사용한다. 이러면 버퍼가 가득 찰 때까지는 유실되는 데이터 없이 통신할 수 있다.

  3) 버퍼 종류
     - 스택 버퍼
       - 스택에 있는 지역 변수
     - 힙 버퍼
       - 힙에 할당된 메모리 영역
      

# 버퍼 오버플로우
  1) 무엇인가?
     - 말 그대로 버퍼가 넘치는 것.
     - 예를 들어, int로 선언한 지역 변수를 4바이트의 크기를 갖는다. 이곳에 4바이트 이상의 데이터를 넣으면 오버플로우가 발생한다.

  2) 왜 문제가 되는가?
     - 일반적으로 **버퍼는 메모리상에 연속해서 할당되어** 있다. 어떤 버퍼에서 오버플로우가 발생하면, 뒤에 있는 버퍼들의 값이 조작될 위험이 있다.
    

# 버퍼 오버플로우 공격 예시
```
// Name: sbof_auth.c
// Compile: gcc -o sbof_auth sbof_auth.c -fno-stack-protector
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_auth(char *password) {
    int auth = 0;
    char temp[16];
    
    strncpy(temp, password, strlen(password));
    
    if(!strcmp(temp, "SECRET_PASSWORD"))
        auth = 1;
    
    return auth;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("Usage: ./sbof_auth ADMIN_PASSWORD\n");
        exit(-1);
    }

    if (check_auth(argv[1]))
        printf("Hello Admin!\n");
    else
        printf("Access Denied!\n");
}
```
  1) 코드 설명
     - main함수는 명령줄 인수로 2개의 값을 받는다. argc는 인수 개수를 의미하고 argv[]는 인수를 저장하는 배열이다.
     - 첫 번째 if문으로 인수가 2개가 아니라면 프로그램을 종료한다. 요구하는 인수는 (1) 프로그램 이름 (2) 사용자가 입력하는 비밀번호이다.
     - 두 번째 if문에서는 check_auth 함수를 호출하여 입력한 비밀번호를 검사한다. 이때, argv[1]로 사용자가 입력한 비밀번호를 전달한다.
     - check_auth함수로 이동. auth변수를 초기화한다. 이 변수는 인증 상태를 나타내며, 0은 인증 실패, 1은 인증 성공을 의미한다.
     - 비밀번호를 비교할 temp문자열 배열을 선언한다. 이 배열은 크기가 16으로 고정되어 있다. 이 부분에서 **버퍼 오버플로우 문제가 생긴다.**
     - strlen() 함수로 길이를 계산하고, strncpy 함수를 사용해 입력한 password를 temp 배열로 복사한다.
     - strcmp() 함수: 문자열이 같으면 0, 다르면 음수 또는 양수를 반환한다. !0이면 if문을 실행하니, 두 문자열이 같으면 if문이 실행되어 auth가 1이 된다. (인증 성공)
     - check_auth() auth값을 반환한다.
     - main 함수로 돌아와 check_auth의 반환 값을 확인한 후, 인증에 성공하면 "Hello Admin!"을 출력하고, 실패하면 "Access Denied!"를 출력합니다.

  2) 문제점
     - strncpy함수로 tmep 버퍼를 복사할 때, argv[1]에 16바이트가 넘는 문자열을 전달하면 이들이 모두 복사되어 스택 버퍼 오버플로우가 발생하게 된다.
     - ※핵심: auth는 temp 버퍼 뒤에 존재하므로, temp 버퍼에 오버플로우를 발생시키면 auth의 값을 0이 아닌 임의의 값으로 바꿀 수 있다. 이 경우, 실제 인증 여부와는 상관없이 main함수의 if (check_auth(argv[1])) 는 항상 참이 된다.
       - 이유: C언어의 if문은 0이 아닌 모든 값을 참(true)으로 평가한다. 그래서 버퍼 오버플로우로 인해 auth값이 0이 아닌 다른 값으로 변경되면 항상 인증이 성공한 것처럼 동작하게 된다.


# C언어의 널 바이트로 인한 데이터 유출
  1) 과정
    - C언어에서 문자열은 \0 (널 바이트)로 끝난다. ex) "hello" 라는 문자열은 메모리에 "h e l l o \0" 처럼 저장된다.
    - C언어의 출력 함수 printf 등은 문자열을 출력할 때 \0을 만날 때까지 출력한다.  ex) "hello\0world" 라는 문자열을 printf로 출력할 때, hello까지만 출력한다.
    - 버퍼 오버플로우가 발생하면 널 바이트도 덮어써질 수 있다.
    - 이런 경우 출력 함수는 문자열이 끝나지 않았다고 판단해 계속 출력하게 된다. 결국 다른 버퍼의 데이터까지 출력된다.
    - 이를 통해 중요한 정보가 유출될 수 있다.


# 실행 흐름 조작
  1) 예시 코드
     ```
      // Name: sbof_ret_overwrite.c
      // Compile: gcc -o sbof_ret_overwrite sbof_ret_overwrite.c -fno-stack-protector
      #include <stdio.h>
      #include <unistd.h>
      
      void win() {
          printf("You won!\n");
      }
      
      int main(void) {
          char buf[8];
          printf("Overwrite return address with %p:\n", &win);
          read(0, buf, 32);
          return 0;
      }
     ```
     - 원래 win() 함수는 출력되지 않음.
     - main함수의 buf는 8바이트 크기의 버퍼이다.
     - read() 는 32바이트의 입력을 받아 buf에 저장한다. 이렇게 되면 버퍼 오버플로우가 발생한다.
     - 공격자는 이 return address를 win() 함수의 주소로 덮어쓰면, main 함수가 끝난 후 원래 반환해야 할 위치 대신 win() 함수로 이동하게 됩니다

  2) 공격 방법
     - 위 코드에서는 win() 의 주소를 출력하고, 8바이트 버퍼 buf에 32바이트 입력을 받습니다. buf 이후에는 saved RBP 값 8바이트와 반환 주소 8바이트가 있으므로, b'A' * 16 이후에 win() 주소를 이어 붙여 보내면 win() 함수를 호출하는 것이 가능합니다.






