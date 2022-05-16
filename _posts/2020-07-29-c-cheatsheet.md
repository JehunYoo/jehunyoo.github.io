---
title: "C CheatSheet"
tags: [c]
categories: [C]
toc: true
math: false
---

## 1D Array

**#1**<br>
``` c
int main(void)
{
    int arr[2];
    arr[0]=0, arr[1]=1, arr[2]=2;
}
```
<div class="notice--info" markdown="1">
&#8594; `arr[2]=2;`<br>
컴파일러는 배열 접근에 있어서 유효성 검사를 진행하지 않는다.<br>
&#8756; compile error 발생 X<br>
할당되지 않은 메모리 공간을 침범할 수 있으므로 주의하자.
</div>

**#2 Length of Array**<br>
``` c
len = sizeof(arr) / sizeof(/* type of element */);
```
<div class="notice--warning" markdown="1">
함수 내부에서는 배열의 길이를 구할 수 없다.<br>
함수 내부에서 argument로 받은 배열의 `sizeof`연산 결과는 64bit 시스템에서는 8byte, 32bit 시스템에서는 4byte이다.<br>
즉, pointer의 크기와 같다.
</div>

**#3 String & Null ('\0')**<br>
<div class="notice--info" markdown="1">
어떤 문자열 `str`에 대해 길이가 `len`이라 하자.<br>
`null`은 `str[len]`에 저장되어 있다.
</div>

## Pointer

**#1 Operator & and \***<br>
<div class="notice--info" markdown="1">
- `&(operand)` : operand의 **주소 값** 반환
- `*(operand)` : operand가 가리키는 주소 **참조**
</div>
<div class="notice--warning" markdown="1">
`&`연산자는 상수를 피연산자로 사용할 수 없다.
</div>


**#2**<br>
``` c
#include <stdio.h>

int main(void)
{
    int * ptr1 = 0;
    int * ptr2 = NULL;
}
```
<div class="notice--info" markdown="1">
두 방식 모두 `NULL` pointer로 초기화 한다.<br>
주소값 0을 갖는 것이 아니라 메모리 아무데도 가리키지 않는다는 의미이다.
</div>

## Pointer & Array

**#1 상수 형태의 pointer**<br>
``` c
int arr[3] = {0, 1, 2};
arr = &arr[i]; // for i in [0, 1, 2]
```
<div class="notice--info" markdown="1">
배열 `arr`는 상수 형태의 pointer 이다.<br>
따라서 `arr`에 저장되어 있는 주소 값을 변경 할 수 없다.<br>
`arr = &arr[i]`는 compile error를 발생시킨다.
</div>

**#2 문자열 상수 & 문자열 변수**<br>
``` c
char str1[] = "Hello, World!";
char * str2 = "Hello, World!";
```
<div class="notice--info" markdown="1">
- `str1` : pointer 상수 &#8594; 문자열 변수 (문자열 변경 가능)
- `str2` : pointer 변수 &#8594; 문자열 상수 (문자열 변경 불가)
</div>

``` c
str1[12] = '?'; // 문자열 변경 가능
str2[12] = '?'; // 문자열 변경 불가
```
<div class="notice--warning" markdown="1">
위의 코드는 컴파일은 되지만 실행은 안된다.<br>
문제가 발생하는 형태는 컴파일러나 컴파일 모드에 따라서 프로그램이 종료되거나 문제가 있는 코드를 무시하는 등 약간씩 차이가 있다.
</div>

**#3 Size of Pointer Array**<br>
``` c
int * arr1[100];
double * arr2[100];
```
<div class="notice--info" markdown="1">
64bit 시스템 (size of pointer = 8byte) 기준
- `sizeof(arr1)` = `sizeof(int *)` &#215; 100 = 800byte
- `sizeof(arr2)` = `sizeof(double *)` &#215; 100 = 800byte

두 배열의 크기가 같다!
</div>

**#4 Array of String**<br>
문자열 하나를 변수 `str`에 저장
``` c
char * str = "Hello, World!";
// OR
char str[] = "Hello, World!";
```

문자열 여러개를 문자열 배열 `str`에 저장
``` c
char * str[] = {"Hello", "World!"};
```
<div class="notice--info" markdown="1">
> 큰따옴표로 묶여서 표현되는 문자열은 그 형태에 상관없이 메모리 공간에 저장된 후 그 **주소 값이 반환**된다.

C에서 문자열을 메모리 주소 값으로 다루기 때문에 문자열 배열 원소의 자료형은 `char *`이다.
</div>

**#5 Array of String 2**<br>
``` c
char c = 'c'; // Character
char * str = "Hello, World!"; // String = Pointer
char arrc[] = {'a', 'b', 'c'}; // Array of Character
char * arrs[] = {"Hello", "World!"}; // Array of String = Array of Pointer
```

<div class="notice--info" markdown="1">
Data Types
- Character : `char`
- String : `char *`
- Array of Character (&#8786; String) : collection of `char`
- Array of String : collection of `char *`
</div>

<div class="notice--warning" markdown="1">
- Array of Characters &#8594; There's no null character.
- String &#8594; There **is** null character.
</div>

## Pointer & Function

**#1**<br>
<div class="notice--info" markdown="1">
C에서 함수의 호출 방식은 기본적으로 call-by-value이다.<br>
pointer를 사용해서 call-by-reference 방식으로 호출하는 것을 메모리 주소 값을 argument로 주어 call-by-value로 호출했다고 생각할 수 있다.<br>
그렇기 때문에 pointer를 argument로 주었다고 해서 혹은 parameter가 pointer라고 해서 무조건 call-by-reference가 아니다.<br>
pointer에 대해 call-by-reference가 되게 하려면 double pointer를 사용해야 한다.
</div>

**#2 배열을 함수의 argument로 전달하는 방법**<br>
``` c
void function1(int arr[]);
void function2(int * arr);
```
<div class="notice--info" markdown="1">
함수의 parameter를 선언할 때에 한해서 `int * arr`을 `int arr[]`로 대체할 수 있다.<br>
`int`가 아닌 다른 자료형도 물론 가능하다.
</div>

**#3**<br>
``` c
void function(int arr[])
{
    int size = sizeof(arr);
    printf("%d\n", size);
}
```
``` bash
8
```
<div class="notice--info" markdown="1">
64bit 기준으로 8byte이다.
</div>

**#4 scanf**<br>
<div class="notice--info" markdown="1">
`scanf` 함수는 call-by-reference 호출 방식을 사용한다.<br>
argument로 메모리 주소 값을 전달해 주어야 하기 때문에 일반적인 변수의 경우 `&` 연산자를 사용한다.
</div>

**#5 const 1**<br>
``` c
int num = 1;
const int * ptr = &num;
*ptr = 100; // compile error
```
<div class="notice--info" markdown="1">
`ptr`에 저장된 메모리 주소 값을 참조하여 그 곳에 저장된 값을 변경하는 것을 허용하지 않는다.<br>
다만 `ptr`을 통해 값을 변경하는 방법만 허용되지 않고 `num`을 통해 값을 변경하는 것은 가능하다.
</div>

**#6 const 2**<br>
``` c
int num1 = 1, num2 = 2;
int * const ptr = &num1;
ptr = &num2; // compile error
```
<div class="notice--info" markdown="1">
`ptr`을 pointer 상수로 만든다.<br>
`int arr[2];`에서의 `arr`과 같은 형태의 pointer 상수이다.<br>
따라서 `ptr`에 다른 주소 값을 대입하면 compile error가 발생한다.
</div>

``` c
int num1 = 1;
const int * const ptr = &num;
```
<div class="notice--info" markdown="1">
위와 같은 선언도 가능하다.<br>
</div>

**#8 const 3**<br>
``` c
void printAllElement(const int * arr, int len)
{
    for(int i=0; i<len; i++)
        printf("%d\n", *(arr + i));
}
```
<div class="notice--info" markdown="1">
`const`의 사용은 프로그램의 안정성을 높여주는 장점이 있다.
</div>

``` c
void printAllElement(const int * arr, int len)
{
    int * ptr = arr; // warning message
    for(int i=0; i<len; i++)
        printf("%d\n", *(arr + i));
}
```
<div class="notice--info" markdown="1">
`int * ptr = arr;`에서 warning message가 발생한다.
</div>

## Multi-Dimensional Array

**#1**<br>
``` c
#include <stdio.h>

int main(void)
{
    int arr[5][5] = {0};

    for (int i=0; i<5; i++) {
        for(int j=0; j<5; j++)
            printf("%d ", arr[i][j]);
        printf("\n");
    }
    return 0;
}
```
```
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
```
<div class="notice--info" markdown="1">
`int arr[5][5] = {0};`와 같은 방법으로 다차원 배열도 한번에 0으로 초기화할 수 있다.
</div>

**#2**<br>
``` c
int arr[3][2];
```
<div class="notice--info" markdown="1">
위와 같이 선언된 3 &#215; 2 배열에 대해 메모리는 다음과 같은 순서로 할당된다.

- &arr[0][0] = 0x7ffee1178b00
- &arr[0][1] = 0x7ffee1178b04
- &arr[1][0] = 0x7ffee1178b08
- &arr[1][1] = 0x7ffee1178b0c
- &arr[2][0] = 0x7ffee1178b10
- &arr[2][1] = 0x7ffee1178b14

`sizeof(int)`의 간격으로 할당되었다.
</div>

**#3**<br>
{% raw %}
``` c
int arr[3][3] = {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}};
int arr[3][3] = {{1}, {4, 5}, {7, 8, 9}};
int arr[3][3] = {1, 2, 3, 4, 5, 6, 7};
```
{% endraw %}
<div class="notice--info" markdown="1">
모두 가능한 초기화 방식이다.<br>
부족한 부분은 0으로 초기화 된다.
</div>

**#4**<br>
``` c
int arr1[][3] = {1, 2, 3, 4, 5, 6};
int arr2[][2] = {1, 2, 3, 4, 5, 6};
```
<div class="notice--info" markdown="1">
배열의 세로 길이만 생략이 가능하다.
</div>

## Double Pointer

**#1**<br>
``` c
**dtpr == *(*dptr); // true
```

**#2**<br>
``` c
#include <stdio.h>

int main(void)
{
    int num1 = 10;
    double num2 = 10.1;

    int * ptr1 = &num1;
    double * ptr2 = &num2;

    int ** dptr1 = &ptr1;
    double ** dptr2 = &ptr2;

    printf("%p %p\n", ptr1, ptr2);
    printf("%p %p\n", dptr1, dptr2);

    return 0;
}
```
``` bash
0x7ffee1984b18 0x7ffee1984b10
0x7ffee1984b08 0x7ffee1984b00
```
<div class="notice--info" markdown="1">
우연히 메모리에 연속적으로 할당되었다.<br>
어찌되었건 pointer의 크기는 8byte이다. (for 64bit system)<br>
pointer의 크기가 8byte임에 착안하여 `ptr1`과 `ptr2`의 주소를 각각 `double **`, `int **`형 double pointer 변수에 대입하면 어떻게 될까?
</div>

``` c
#include <stdio.h>

int main(void)
{
    int num1 = 10;
    double num2 = 10.1;

    int * ptr1 = &num1;
    double * ptr2 = &num2;

    int ** dptr1 = &ptr1;
    double ** dptr2 = &ptr2;

    double ** dptr11 = &ptr1;
    int ** dptr22 = &ptr2;

    printf("%p %p\n", ptr1, ptr2);
    printf("%p %p\n", dptr1, dptr2);
    printf("%p %p\n", dptr11, dptr22);
    printf("%p %p\n", *dptr11, *dptr22);

    return 0;
}
```
``` bash
0x7ffee1984b18 0x7ffee1984b10
0x7ffee1984b08 0x7ffee1984b00
0x7ffee1984b08 0x7ffee1984b00
0x7ffee1984b18 0x7ffee1984b10
```
<div class="notice--info" markdown="1">
`int *`, `double *` 모두 8byte의 크기이기 때문에 결과가 같게 나오는 것 같다.<br>
그럼 `num1`, `num2`의 값을 참조해보면 어떻게 될까?
</div>

``` c
printf("%d %f\n", **dptr11, **dptr22); // warning message
printf("%f %d\n", **dptr11, **dptr22);
```
``` bash
858993459 0.000000
0.000000 858993459
```
<div class="notice--info" markdown="1">
두 문장을 추가했다.<br>
물론 `num1`과 `num2`의 값을 올바르게 참조할 수는 없다.<br>
흥미로운 점은 첫번째 줄에서만 warning이 발생했다는 것이다.<br>
어찌보면 강제로 주소 값을 type casting 시켰기 때문에 당연한 결과일 수 있다.<br>
그냥 궁금해서 해봤는데 이런 방법은 쓰지 말자.
</div>

**#3 Type of Array**<br>
``` c
int arr1[10];       // type of arr1  = int *
double arr2[10];    // type of arr2  = double *
int * arr11[10];    // type of arr11 = int **
double * arr22[10]; // type of arr22 = double **
```

## Pointer & Multi-Dimensional Array

**#1**<br>
``` c
#include <stdio.h>

int main(void)
{
    int arr[2][3];

    printf("%p\n", arr);
    printf("%p\n", arr[0]);
    printf("%p\n", &arr[0][0]);
    return 0;
}
```
``` bash
0x7ffee3a7cb00
0x7ffee3a7cb00
0x7ffee3a7cb00
```
<div class="notice--info" markdown="1">
`arr`, `arr[0]`, `&arr[0][0]` 모두 같은 메모리 주소 값을 가리킨다.<br>
`arr`, `arr[0]`은 그 자체로 pointer이다.<br>
`arr[0][0]`은 index (0, 0)의 원소 값을 가리킨다.<br>
index (0, 0)의 원소의 메모리 주소는 `&arr[0][0]`이다.
</div>

**#2**<br>
``` c
int arr[2][3];
```
<div class="notice--info" markdown="1">
- `sizeof(arr)` = `sizeof(int)` &#215; 2 &#215; 3 = 24byte
- `sizeof(arr[0])` = `sizeof(int)` &#215; 3 = 12byte

`arr`은 `arr[0][0]`의 주소를 가리키면서 배열 전체를 의미한다.<br>
`arr[0]`은 `arr[0][0]`의 주소를 가리키면서 index 0행만을 의미한다.
</div>

**#3 `arr + 1`**<br>
``` c
int arr1[2][3];
double arr2[2][3];

printf("%p\n", arr1);
printf("%p\n", arr1 + 1);
printf("%p\n", arr2);
printf("%p\n", arr2 + 1);
```
``` bash
0x7ffee053fb00
0x7ffee053fb0c
0x7ffee053fad0
0x7ffee053fae8
```
<div class="notice--info" markdown="1">
- &#8710;`arr1` = `arr1 + 1` - `arr1` = `sizeof(int)` &#215; 3 = 12byte
- &#8710;`arr1` = `arr2 + 1` - `arr2` = `sizeof(int)` &#215; 3 = 24byte

배열 이름에 1을 더하면 `sizeof(/* element type */)` &#215; `(한 행의 element 개수)`byte 만큼 주소값이 더해진다.
</div>

**#4**<br>
``` c
arr[i][j] == *(*(arr + i) + j) // true
```

**#5 Type of Multi-Dimensional Array**<br>
``` c
element arr[?][num];
```
<div class="notice--info" markdown="1">
`arr`은 가리키는 대상이 `element`형 변수이고, pointer 연산 시 (`sizeof(element)` &#215; `num`)byte로 증감하는 pointer type이다.
</div>

**#6 Declaration**<br>
``` c
int arr[?][num];
int (*ptr)[num] = arr;

int arr[2][3];
int (*ptr)[3] = arr;
```
<div class="notice--info" markdown="1">

앞에서 언급했듯이 다차원 배열에서 `arr[0]`도 pointer이다.<br>
`arr[0]`도 하나의 배열이라고 가정하면, 다음과 같이 생각해볼 수 있겠다.
</div>

``` c
element (*ptr)[/* size of array arr[0] */] = arr;
```

**#7 Array of Pointer v. Pointer to Array**<br>
``` c
int *arr[10]; // Array of Pointer

int arr[?][10];
int (*ptr)[10] = arr; // Pointer to Array
```
<div class="notice--warning" markdown="1">
- Array of Pointer (포인터 배열) : pointer로 이루어진 배열
- Pointer to Array (배열 포인터) : 배열을 가리키는 pointer = `arr`, `ptr`
</div>

**#8 Argument about Multi-Dimensional Array**
``` c
void function(int (*arr)[3]);
// OR
void function(int arr[][3]);

int arr[2][3];
function(arr);
```

**#9 `*(arr + i)` & `arr + i`**<br>
``` c
int arr[2][3] = {1, 2, 3, 4, 5, 6};

printf("%p %p %p\n", arr + 1, arr[1], &arr[1][0]);
printf("%p %p\n", *(arr + 1), &arr[1]);
```
``` bash
0x7ffee9e2bb0c 0x7ffee9e2bb0c 0x7ffee9e2bb0c
0x7ffee9e2bb0c 0x7ffee9e2bb0c
```
<div class="notice--info" markdown="1">
모두 같은 주소 값을 갖고 있다.<br>
특히 `arr + 1`과 `*(arr + 1)`이 같은 값을 갖고 있다는 것이 놀라웠다.
</div>

``` c
printf("%d\n", *(arr + 1)); // warning message
```
<div class="notice--info" markdown="1">
위의 문장은 warning을 발생시킨다.<br>
또한 warning으로부터 `*(arr + 1)`의 타입이 `int *`라는 사실을 알게되었다.
</div>

``` c
printf("%d %d\n", arr[1][2], *((arr + 1) + 2)); // warning message
```
<div class="notice--info" markdown="1">
따라서 두 값은 다른 값으로 출력된다.
</div>

<div class="notice--warning" markdown="1">
그럼 `*(arr + 1)`과 `arr + 1`의 차이가 뭘까?<br>
일부러 warning을 발생시켜 보니 다음을 알 수 있었다!

- `*(arr + 1)` : type `int *`
- `arr + 1` &nbsp; &nbsp; &nbsp; : type `int (*)[3]`

`*(arr + 1)`과 `arr + 1`은 가리키는 주소 값은 같지만 type이 다르다.<br>
`arr + 1`은 `arr`과 같은 `int (*)[3]` type이다.<br>
반면에 `*(arr + 1)`은 `arr[1]`과 같은 `int *` type이다.
</div>


**#10 `arr[i] == *(arr + i)`**<br>
``` c
int arr[][]; // any 2D array
arr[i] == *(arr + i); // true
```
<div class="notice--info" markdown="1">
앞에서 언급했듯이 다차원 배열에서 `arr[i]`는 pointer이다.<br>
`arr[i]`는 `&arr[i][0]`와 같은 주소를 가리킨다.

`arr[i]`가 pointer이므로 `arr[i]`를 어떤 배열 `arri`라 하자.<br>
`arri`는 arr의 i행의 원소들이 순서대로 들어가있다.<br>
따라서 배열 `arri`의 0번째 원소는 `*arri`이고 그 값은 `arr[i][0]`과 같다.

`arri`의 타입은 `int *`일 것이다.<br>
따라서 배열 `arri`의 1번째 원소는 `*(arri + 1)`이 된다.<br>
또한 이때의 값은 `arr[i][1]`, `arri[1]`과 같다.<br>

이때 `arri`는 `arr[i]`이고 `*(arr + i)`와 같다.
</div>
<div class="notice--warning" markdown="1">
여기서 `*(arr + i)`는 `arr`의 원소가 아니다.<br>
`*(arr + i)`는 주소를 가리키고 `int *` type이다.
</div>

**#11**<br>
``` c
double * arr1[5];    // 1d array of pointer of double * type
double * arr2[3][5]; // 2d array of pointer of double * type

double **ptr1 = arr1;
double * (*ptr2)[5] = arr2;
```

## Function Pointer & Void Pointer

**#1**<br>
``` c
int function(int param);
```
<div class="notice--info" markdown="1">
함수도 프로그램 실행 시에 메모리에 저장되어서 실행된다.<br>
따라서 함수의 이름 `function`은 함수가 저장된 메모리 주소 값을 의미한다.<br>
이때 `function`은 배열의 이름과 같이 pointer 상수이다.
</div>

**#2 Function Pointer 변수**<br>
``` c
return_type function(param_type1 param1, param_type2 param2);
return_type (*fptr)(param_type1, param_type2) = function;
```
<div class="notice--info" markdown="1">
function pointer 선언과 초기화 방법이다.
</div>

``` c
double function(int a, int *b);

int a = 10;
int *b = &a;

double (*fptr)(int, int *) = function;
fptr(a, b) == function(a, b) // true
```
<div class="notice--info" markdown="1">
이를 이용하면 함수의 parameter로 function pointer 변수를 사용할 수 있다.
</div>

``` c
void function2(double (*fptr)(int, int *));

function2(fptr);
```

**#3 Void Pointer**<br>
``` c
void *ptr;
```
<div class="notice--info" markdown="1">
void pointer는 변수의 종류에 상관없이 변수의 주소 값을 저장할 수 있다.<br>
그렇지만 type에 대한 정보가 없으므로 주소 값을 참조할 수 없다.
</div>

**#4 `main` Function**<br>
``` c
int main(void) {}
// OR
int main(int argc, char * argv[]) {}
```
<div class="notice--info" markdown="1">
`main`함수의 parameter를 위와 같이 할 수도 있다.<br>
`argc`에는 argument의 개수가 저장된다. (파일 이름 포함)<br>
이는 아마 함수 내부에서 배열 `argv`의 길이를 구할 수 없기 때문에 `main`함수 외부에서 계산해서 넘겨주는 것 같다.<br>
`argv`에는 파일 이름도 포함된다.
</div>


## Variables

**#1 Local Variables**<br>
<div class="notice--info" markdown="1">
- 중괄호 내에 선언되는 변수
- memory의 stack 영역에 할당
- parameter: local variable
</div>

**#2 Local Variables**<br>
``` c
#include <stdio.h>

int main(void)
{
    int i = 100;
    for(int i=0; i<10; i++)
        continue;
    printf("%d\n", i);
}
```
``` bash
100
```

``` c
#include <stdio.h>

int main(void)
{
    for(int i=0; i<10; i++)
        continue;
    printf("%d\n", i);
}
```
``` bash
error: use of undeclared identifier 'i'
printf("%d\n", i);
               ^
```
``` c
#include <stdio.h>

int main(void)
{
    int i;
    for(i=0; i<10; i++)
        continue;
    printf("%d\n", i);
}
```
``` bash
10
```
<div class="notice--info" markdown="1">
변수 `i`의 범위에 유의하자.<br>
또한 `for`에 의한 반복은 중괄호의 진입과 탈출을 반복하면서 이뤄진다.
</div>

**#3 Global Variables**<br>
<div class="notice--info" markdown="1">
- 중괄호 내에서 선언되지 않는다.
- 별도의 값으로 초기화 하지 않으면 0으로 초기화된다.
- memory의 data 영역에 할당
- 프로그램의 시작과 동시에 할당되어 종료 시까지 존재한다.
- 같은 이름의 지역 변수가 지역 변수를 가리킨다.
</div>

**#4 Static Variables**<br>
<div class="notice--info" markdown="1">
- `static` 키워드로 선언할 수 있다.
- 전역 변수, 지역 변수, 함수에 `static` 선언을 할 수 있다.
- 전역 변수와 함수에 `static` 선언을 하면 외부 파일에서의 접근을 허용하지 않는다.
- 지역 변수 특성 : 선언된 함수 내에서만 접근이 가능하다.
- 전역 변수 특성 : 딱 1회 초기화되고 프로그램 종료 시까지 메모리 공간에 존재한다.
- 전역 변수와 마찬가지로 memory의 data 영역에 할당
- static 변수는 접근 범위를 제한할 수 있기 때문에 전역 변수 보다 안정적이다.
</div>

``` c
#include <stdio.h>

void function()
{
    static int var1 = 0; // 접근 범위를 function으로 제한 / 초기화는 처음 1번만, 이후에는 이 줄이 무시되고 메모리에 저장된 값 사용
    int var2 = 0;
    printf("%d %d\n", var1++, var2++);
}

int main(void)
{
    for(int i=0; i<3; i++)
        function();
    return 0;
}
```
``` bash
0 0
1 0
2 0
```

**#5 Register Variables**<br>
<div class="notice--info" markdown="1">
- Register : CPU 내에 존재하는 크기가 매우 작은 메모리 - Register 변수에 대한 연산이 매우 빠르다.
- 전역 변수에는 `register` 선언을 할 수 없다.
- `register` 선언을 해도 컴파일러가 합당하지 않다고 판단하면 register에 할당하지 않는다.
- 반대로 `register` 선언을 하지 않아도 컴파일러가 필요하다고 판단하면 register에 할당한다.
</div>



## Reference
- [열혈 C 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=6393451)
