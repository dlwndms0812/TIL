

- 동적 메모리 할당: 동적으로 메모리 할당

```sql
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
  int SizeOfArray;
  int *arr;

  printf("만들고 싶은 배열의 원소의 수 : ");
  scanf("%d", &SizeOfArray);

  arr = (int *)malloc(sizeof(int) * SizeOfArray);
  // int arr[SizeOfArray] 와 동일한 작업을 한 크기의 배열 생성

  free(arr);

  return 0;
}
```

- malloc
    - memry allocation의 약자
    - #include <stdlib.h> 추가
    - 인자로 전달된 크기의 바이트 수 만큼 메모리 공간 만듦
    - 힙(heap)을 이용

원소의 개수가 SizeOfArray인 int형 배열을 만들기 위해서는 (int의 크기)*(SizeOfArray)를 해야함

⇒ 리턴형이 (void *)형이므로 (int *)형으로 형변환 하여 arr에 넣음

- free
    - 할당 받은 메모리를 다시 컴퓨터에게 돌려주는 것 = ‘해제’
    - free를 제대로 하지 않으면 메모리 누수 발생
    

### 2차원 배열의 동적 할당

1. **포인터 배열을 이용해서 2차원 배열 할당하기**

```sql
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
  int i;
  int x, y;
  int **arr;  // 우리는 arr[x][y] 를 만들 것이다.

  printf("arr[x][y] 를 만들 것입니다.\n");
  scanf("%d %d", &x, &y);

  arr = (int **)malloc(sizeof(int *) * x);
  // int* 형의 원소를 x 개 가지는 1 차원 배열 생성

  for (i = 0; i < x; i++) {
    arr[i] = (int *)malloc(sizeof(int) * y);
  }

  printf("생성 완료! \n");

  for (i = 0; i < x; i++) {
    free(arr[i]);
  }
  free(arr);

  return 0;
}
```

![Untitled](https://user-images.githubusercontent.com/57864944/160835553-a919abce-bade-4c46-b575-0c9fb3946d16.png)


- arr원소들이 가리키는 메모리 공간이 연달아 존재X
- 2차원 배열이 아닌 int* 형을 가지는 1차원 배열
- 2차원 배열의 성질을 띔

1. **실제로 2차원 배열 크기의 메모리를 할당한 뒤 2차원 배열 포인터로 참조하는 방법**
- malloc을 통해 해당 크기의 공간을 할당해야 메모리에 연속적으로 존재하는 2차원 배열을 만들 수 있음

```sql
#include <stdio.h>
#include <stdlib.h>

int main() {
  int width, height;
  printf("배열 행 크기 : ");
  scanf("%d", &width);
  printf("배열 열 크기 : ");
  scanf("%d", &height);

  int(*arr)[width] = (int(*)[width])malloc(height * width * sizeof(int));
  for (int i = 0; i < height; i++) {
    for (int j = 0; j < width; j++) {
      int data;
      scanf("%d", &data);
      arr[i][j] = data;
    }
  }
  for (int i = 0; i < height; i++) {
    for (int j = 0; j < width; j++) {
      printf("%d ", arr[i][j]);
    }
    printf("\n");
  }

  free(arr);
}
```

⇒ 2번의 방법이 더 좋음

- malloc의 호출 횟수를 최소한으로 하는 것이 좋음
- 메모리가 연속적으로 있을 경우 접근이 더 빠름

### 구조체 동적 할당

```sql
#include <stdio.h>
#include <stdlib.h>
struct Something {
  int a, b;
};
int main() {
  struct Something *arr;
  int size, i;

  printf("원하시는 구조체 배열의 크기 : ");
  scanf("%d", &size);

  arr = (struct Something *)malloc(sizeof(struct Something) * size);

  for (i = 0; i < size; i++) {
    printf("arr[%d].a : ", i);
    scanf("%d", &arr[i].a);
    printf("arr[%d].b : ", i);
    scanf("%d", &arr[i].b);
  }

  for (i = 0; i < size; i++) {
    printf("arr[%d].a : %d , arr[%d].b : %d \n", i, arr[i].a, i, arr[i].b);
  }

  free(arr);

  return 0;
}
```

- 구조체는 사용자가 만든 하나의 데이터 타입이니 malloc 사용에 이상 없음

### 노드

```sql
struct Node {
  int data;              /* 데이터 */
  struct Node* nextNode; /* 다음 노드를 가리키는 부분 */
};
```

- 노드를 사용하면 기존의 배열에서 불가능 한 ‘배열 중간에 새 원소 집어넣기’가 가능

```
/* 새 노드를 만드는 함수 */
struct Node* CreateNode(int data) {
  struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));

  newNode->data = data;
  newNode->nextNode = NULL;

  return newNode;
}

/* current 라는 노드 뒤에 노드를 새로 만들어 넣는 함수 */
struct Node* InsertNode(struct Node* current, int data) {
  /* current 노드가 가리키고 있던 다음 노드가 after 이다 */
  struct Node* after = current->nextNode;

  /* 새로운 노드를 생성한다 */
  struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));

  /* 새 노드에 값을 넣어준다. */
  newNode->data = data;
  newNode->nextNode = after;

  /* current 는 이제 newNode 를 가리키게 된다 */
  current->nextNode = newNode;

  return newNode;
}
```

**reference**
[씹어먹는 C 언어](https://modoocode.com/98)
