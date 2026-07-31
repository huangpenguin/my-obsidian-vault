---
title: "C语言指针代码留档"
publish: false
tags: ["C++"]
---
# C语言指针代码留档

```cpp
/*配列の添字（そえじ）えんざんしのオペランドの順序が任意*/
void set_idx(int *v, int n)
//void set_idx(int v[], int n)同じ
{
	for (int i = 0; i < n; i++)
		//v[i] = i;
		//*(v + i) = i;
		i[v] = i;
}

int main(void)
{
	int a[100];

	set_idx(a, 100);

	for (int i = 0; i < 100; i++) {
		printf("a[%d] = %d", i, a[i]);
		putchar(' ');
		puts("");
	}

	return 0;

}
```

```cpp
#include <stdio.h>
int main(void)
{
	char a = '5';
	const char *p = "123";

	printf("p = \"%s\"\n", p);

	p = "456" ;

	printf("p = \"%s\"\n", p);

	return 0;
}
```

```cpp
/*問： strcpy関数およびstrncpy関数と同じ仕様の関数を作成せよ。*/
#include <stdio.h>

char *scpy(char *s1, const char *s2, size_t n)
{
	char *tmp = s1;

	while (n) {

		if (!(*s1++ = *s2++)) break;
		n--;
	}

	while (n--)
		*s1++ = '\0';

	return tmp;
}

int main(void)
{
	char str1[128] = "1234567";// コピー先の文字列
	char str2[128]="abcde";//コピー元の文字列
	int n=3;//先頭から何文字をコピーしますか
	printf("コピー結果：\"%s\"\n", scpy(str1, str2, (size_t)n));
	return 0;
}
```

```cpp
//問： strcat関数およびstrncat関数と同じ仕様の関数を作成せよ。
#include <stdio.h>

char *strn_cat(char *s1,char *s2, int n)
{
	char *tmp = s1;//先把指针存起来

	//while (*s1)
	//s1++;
	while (*s1++);//这里和上面的区别就是这里无论如何到最后会多自增一次，所以跳出while后要做一次自减才是/0的位置
    s1--;
	while (n) {
		*s1++ = *s2++;
		n--;
	}

	return tmp;
}

int main(void)
{
	char str1[10] = "ABC";
	char str2[128]="adc";
	int n=2;

	printf("連結の結果：\"%s\"\n", strn_cat(str1, str2, n));

	return 0;
}

```

```cpp
//問： 文字列sを表示する関数を作成せよ。添字演算子[]を使わずに実現すること。
#include <stdio.h>
#define NUMBER 128

void put_string(const char* s)
{
	while (*s)
		putchar(*s++);

	puts("");
}

int main(void)
{
	const char* str="abcdefg";

	put_string(str);

	return 0;
}
```
