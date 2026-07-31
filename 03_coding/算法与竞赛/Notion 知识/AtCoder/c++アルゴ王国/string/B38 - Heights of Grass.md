---
title: "B38 - Heights of Grass"
publish: false
tags: ["待整理"]
---
# B38 - Heights of Grass

[https://atcoder.jp/contests/tessoku-book/tasks/tessoku_book_dk](https://atcoder.jp/contests/tessoku-book/tasks/tessoku_book_dk)

问题实质：找某一个项正前方的A项的数量和

改进后的代码

```cpp
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;
 
int N, a_fore[1 << 18], b_back[1 << 18];
string S;
 
int main() {
	// 入力
	cin >> N >> S;
 
	// 答えを求める
	int streak1 = 0; a_fore[1] = 0; // streak1 は「何個 A が連続したか」
	for (int i = 1; i < N; i++) {//以后都采用逻辑上的序数//此处扫描了n-1个间隔
		if (S[i-1] == 'A') streak1 += 1;//这里由于数组的序数比逻辑上第i个要少1所以要减1
		if (S[i-1] == 'B') streak1 = 0;
		a_fore[i+1] = streak1;
		cout<<'a'<<i+1<<a_fore[i+1]<<endl;
	}
	int streak2 = 0; b_back[N] = 0; // streak2 は「何個 B が連続したか」
	for (int i = N - 1; i >= 1; i--) {
		if (S[i-1] == 'B') streak2 += 1;
		if (S[i-1] == 'A') streak2 = 0;
		b_back[i] = streak2;
		cout<<'b'<<i<<b_back[i]<<endl;
	}
 
	// 出力
	long long Answer = 0;
	for (int i = 1; i <= N; i++) Answer += max(a_fore[i]+1, b_back[i]+1);//and 
	cout << Answer << endl;
	return 0;
}
```

书上源代码

```cpp
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;
 
int N, ret1[1 << 18], ret2[1 << 18];
string S;
 
int main() {
	// 入力
	cin >> N >> S;
 
	// 答えを求める
	int streak1 = 1; ret1[0] = 1; // streak1 は「何個 A が連続したか」+ 1
	for (int i = 0; i < N - 1; i++) {
		if (S[i] == 'A') streak1 += 1;
		if (S[i] == 'B') streak1 = 1;
		ret1[i + 1] = streak1;
	}
	int streak2 = 1; ret2[N - 1] = 1; // streak2 は「何個 B が連続したか」+ 1
	for (int i = N - 2; i >= 0; i--) {
		if (S[i] == 'B') streak2 += 1;
		if (S[i] == 'A') streak2 = 1;
		ret2[i] = streak2;
	}
 
	// 出力
	long long Answer = 0;
	for (int i = 0; i < N; i++) Answer += max(ret1[i], ret2[i]);
	cout << Answer << endl;
	return 0;
}
```
