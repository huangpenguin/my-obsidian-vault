---
title: "A39 interval scheduling"
publish: false
tags: ["待整理"]
---
# A39 interval scheduling

```cpp
//贪心算法,T=O(answer*N)

#include <iostream>
#include <string>
#include <algorithm>
using namespace std;
 
int N;
int l[300009],r[300009];
int main() {
	// 入力
	cin>>N;
	for(int i=1;i<=N;i++)cin>>l[i]>>r[i];
 
	// 答えを求める
	int CurrentTime=0,ans=0;
	while(true){
		int min_endtime=999999;
		for (int i = 1; i <= N; i++) {
			if(CurrentTime>l[i])continue;
			min_endtime=min(min_endtime,r[i]);
		}
		//扫描完毕后加上该部电影
		if(min_endtime==999999)break;
		CurrentTime=min_endtime;
		ans+=1;
	}
	// 出力
	cout<<ans<<endl;
	return 0;
}
		
		
		
```

改进使用内置的sort函数先升序排序o（nlogn）

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int N, L[300009], R[300009];
vector<pair<int, int>> tmp; // //pair类型

int main() {
	// 入力
	cin >> N;
	for (int i = 1; i <= N; i++) {
		cin >> L[i] >> R[i];
		tmp.push_back(make_pair(R[i], L[i]));//往最后添加
	}

	// R の小さい順にソート
	sort(tmp.begin(), tmp.end());
	for (int i = 1; i <= N; i++) {
		R[i] = tmp[i - 1].first;
		L[i] = tmp[i - 1].second;
	}

	// 終了時刻の早いものから貪欲に取っていく（CurrentTime は現在時刻）
	int CurrentTime = 0, Answer = 0;
	for (int i = 1; i <= N; i++) {
		if (CurrentTime <= L[i]) {
			CurrentTime = R[i];
			Answer += 1;
		}
	}
	cout << Answer << endl;
	return 0;
}
```
