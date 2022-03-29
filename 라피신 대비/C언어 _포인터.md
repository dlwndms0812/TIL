# C언어 포인터

- **포인터**
    - 특정한 데이터가 저장된 주소값을 보관하는 변수
    - (포인터에 주소값이 저장되는 데이터의 형)* (포인터의 이름);
    - (포인터에 주소값이 저장되는 데이터의 형) *(포인터의 이름);

- **&**
    - 피 연산자의 주소값을 불러옴
    
-  ### *
    - 주소값에서 해당 주소값에 대응되는 데이터를 가져옴

![Untitled](https://user-images.githubusercontent.com/57864944/160567515-e5ed9870-6c16-4a42-8680-11fce133470a.png)

```cpp
#include <stdio.h>
int main() {
  int *p;
  int a;

  p = &a;
  a = 2;

  printf("a의 값 : %d \n", a);
  printf("*p의 값 : %d \n", *p);

  return 0;
}

//<결과>
//a의 값: 2
//*p의 값: 2

```

⇒ *p와 변수 a는 동일 (p에 저장된 주소(변수 a의 주소)에 해당하는 데이터 a 그자체) 

```cpp
#include <stdio.h>
int main() {
  int *p;
  int a;

  p = &a;
  *p = 3;

  printf("a 의 값 : %d \n", a);
  printf("*p 의 값 : %d \n", *p);

  return 0;
}

//<결과>
//a의 값:3
//*p의 값:3
```

### 상수 포인터

```cpp
#include <stdio.h>
int main() {
  int a;
  int b;
  const int* pa = &a;

  *pa = 3;  // 올바르지 않은 문장
  pa = &b;  // 올바른 문장
  return 0;
}

//<결과>
// 컴파일 오류 발생
```

- const int* pa=&a
    - int형 변수를 가리키는데 그 값을 절대로 변경X

```cpp
#include <stdio.h>
int main() {
  int a;
  int b;
  int* const pa = &a;

  *pa = 3;  // 올바른 문장
  pa = &b;  // 올바르지 않은 문장

  return 0;
}

//<결과>
//컴파일 오류 발생
```

- int* const pa=&a
    - pa의 값이 변경되면 X

### 배열과 포인터

```cpp
#include <stdio.h>
int main() {
  int arr[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
  int* parr;
  int i;
  parr = &arr[0];

  for (i = 0; i < 10; i++) {
    printf("arr[%d] 의 주소값 : %p ", i, &arr[i]);
    printf("(parr + %d) 의 값 : %p ", i, (parr + i));

    if (&arr[i] == (parr + i)) {
      /* 만일 (parr + i) 가 성공적으로 arr[i] 를 가리킨다면 */
      printf(" --> 일치 \n");
    } else {
      printf("--> 불일치\n");
    }
  }
  return 0;
}

//<결과>
//모두 일치로 나옴
```

⇒ &arr[i] 와 parr+i 은 동일

```cpp
#include <stdio.h>
int main() {
  int arr[3] = {1, 2, 3};

  printf("arr 의 정체 : %p \n", arr);
  printf("arr[0] 의 주소값 : %p \n", &arr[0]);

  return 0;
}

//<결과>
//arr와 &arr[0]의 값이 같음
```

⇒ arr와 arr[0]은 동일

- 배열은 배열이고 포인터는 포인터이지만,
    - sizeof와 주소값 연산자와 함께 사용할 때를 제외하면, 배열의 이름은 첫 번째 원소를 가리킴
    - arr[i]와 같은 문장은 사실 컴파일러에 의해 *(arr+i)로 변환됨

### 1차원 배열 가리키기

```cpp
#include <stdio.h>
int main() {
  int arr[3] = {1, 2, 3};
  int *parr;

  parr = arr;
  /* parr = &arr[0]; 도 동일하다! */

  printf("arr[1] : %d \n", arr[1]);
  printf("parr[1] : %d \n", parr[1]);
  return 0;
}

//<결과>
//arr[1]:2
//parr[1]:2
```

⇒ parr을 통해 arr을 이용했을 때와 동일하게 배열의 원소에 마음껏 접근할 수 있음

### 포인터의 포인터

```cpp
#include <stdio.h>
int main() {
  int a;
  int *pa;
  int **ppa;

  pa = &a;
  ppa = &pa;

  a = 3;

  printf("a : %d // *pa : %d // **ppa : %d \n", a, *pa, **ppa);
  printf("&a : %p // pa : %p // *ppa : %p \n", &a, pa, *ppa);
  printf("&pa : %p // ppa : %p \n", &pa, ppa);

  return 0;
}

//<결과>
//a : 3 // *pa : 3 // **ppa : 3 
//&a : 0x7ffd26a79dd4 // pa : 0x7ffd26a79dd4 // *ppa : 0x7ffd26a79dd4 
//&pa : 0x7ffd26a79dd8 // ppa : 0x7ffd26a79dd8
```

### 배열 이름의 주소값

```cpp
#include <stdio.h>

int main() {
  int arr[3] = {1, 2, 3};
  int (*parr)[3] = &arr;

  printf("arr[1] : %d \n", arr[1]);
  printf("parr[1] : %d \n", (*parr)[1]);
}

//<결과>
//arr[1] : 2 
//parr[1] : 2
```

⇒ 꼭 괄호()를 쳐야함

⇒ parr와 arr은 같은 값을 가짐

### 2차원 배열의 [] 연산자

```cpp
#include <stdio.h>
int main() {
  int arr[2][3];

  printf("arr[0] : %p \n", arr[0]);
  printf("&arr[0][0] : %p \n", &arr[0][0]);

  printf("arr[1] : %p \n", arr[1]);
  printf("&arr[1][0] : %p \n", &arr[1][0]);

  return 0;
}

//<결과>
//arr[0] : 0x7ffda354e530 
//&arr[0][0] : 0x7ffda354e530 
//arr[1] : 0x7ffda354e53c 
//&arr[1][0] : 0x7ffda354e53c
```

⇒ arr[0]은 arr[0][0]을 가리키는 포인터로 암묵적으로 타입 변환되고, arr[1]은 arr[1][0]을 가리키는 포인터로 타입 변환됨

### 포인터 배열

- 포인터 배열: 포인터들의 배열
- 배열 포인터: 배열을 가리키는 포인터

```cpp
#include <stdio.h>
int main() {
  int *arr[3];
  int a = 1, b = 2, c = 3;
  arr[0] = &a;
  arr[1] = &b;
  arr[2] = &c;

  printf("a : %d, *arr[0] : %d \n", a, *arr[0]);
  printf("b : %d, *arr[1] : %d \n", b, *arr[1]);
  printf("b : %d, *arr[2] : %d \n", c, *arr[2]);

  printf("&a : %p, arr[0] : %p \n", &a, arr[0]);
  return 0;
}

//<결과>
//a : 1, *arr[0] : 1 
//b : 2, *arr[1] : 2 
//b : 3, *arr[2] : 3 
//&a : 0x7ffe8a2fa4e4, arr[0] : 0x7ffe8a2fa4e4
```

**reference**

[씹어먹는 C 언어](https://modoocode.com/23)
