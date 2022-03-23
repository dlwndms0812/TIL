### 터미널 사용방법

**커널(kernel)이란?**

- 모든 OS는 커널을 가지고 있음
- 커널은 OS계층이며 하드웨어와 컴퓨터에서 돌아가는 프로그램들을 연결해주는 역할
- 메모리 공간의 낭비를 방지하기 위해 운영체제 중 항상 필요한 부분만을 전원이 켜짐과 동시에 메모리에 올려놓고 그렇지 않은 부분은 필요할 때 메모리에 올려서 사용함
- 메모리에 상주하는 운영체제의 부분으로써 운영체제의 핵심적인 부분

**쉘(Shell)이란?**

- 커널과 사용자 사이에 존재
- 사용자의 명령어를 해석하고 운영체제가 알아들을 수 있게 지시

종류

- sh(Bounce Shell)
    - 프롬프트: $(일반 유저) #(root 유저)
    - 가장 오랜 기간동안 UNIX 시스템의 표준 셀로 이용
    - 상호대화형 방식X
    - /bin/sh와 /sbin/sh가 셀의 절대경로
    - ksh, zsh, bash 등이 이 계열

- bash(Bourne-Again Shell)
    - 프롬프트: #
    - Bounce Shell의 변종
    - 리눅스에서 기본으로 지원되는 쉘로, 사용자 계정을 생성할 때 특별한 셀을 지정하지 않으면 기본적으로 bash 셀로 지정
    - /bin/bash
    
- csh(C프로그램 스타일 Shell)
    - 프롬프트: %
    - C언어와 유사한 언어를 사용
    - 상호 대화형 방식
    - Bounce Shell과 대부분 호환되며, 명령형 편집기능X
    
- ksh(Korn Shell)
    - 프롬프트: $
    - 유닉스에서 가장 많이 사용되고 있는 셀
    - Bounce Shell에 C Shell로부터 차용한 현대적 기능을 도입한 셀

**reference**

[[Backend] 터미널 사용방법](https://nam-ki-bok.github.io/backend/Backend_6/)

[Shell 이란? : Shell의 정의와 종류](https://byebyeblue.tistory.com/5)
