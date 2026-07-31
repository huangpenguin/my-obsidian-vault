---
title: "input(),print()之类的IO"
publish: false
tags: ["待整理"]
---
# input(),print()之类的IO

```python
a=int(input())
b,c=map(int,input().split())
string=input()

a=float(input())
a,b,c=input().split(":")

for i in range(N):
for i in range(N-1,-1,-1):
	a=1
else:
	print("a")

a=list(range(10))

a=[1,6]+[2.3]
a+=[0,3]
print(a.pop())
a.append(1)

a.index(1)
a.count(1)
a.sort()#有函数sorted(a)
a.reverse()#有函数reversed(L)
print(a[0:5])

s.isupper()

s='atcoder'
a=list(s)
a[0]="M"
print("".join(a))

pow(a,b,mod)
min(a)
max(a)
sum(a)

```

```python
print(a[2:5])    # [2, 3, 4]（索引 2~4，不包含 5）
print(a[:3])     # [0, 1, 2]（从头到索引 2）
print(a[5:])     # [5, 6, 7, 8, 9]（索引 5 到末尾）
print(a[:])      # [0, 1, 2, ..., 9]（完整列表的浅拷贝）

print(a[-3:])    # [7, 8, 9]（倒数 3 个元素）
print(a[-5:-2])  # [5, 6, 7]（倒数第 5 个到倒数第 3 个）
print(a[::2])    # [0, 2, 4, 6, 8]（每隔 1 个取 1 个）
print(a[1::2])   # [1, 3, 5, 7, 9]（从索引 1 开始，每隔 1 个取 1 个）
print(a[::-1])   # [9, 8, 7, ..., 0]（反转列表）
print(a[5:2:-1]) # [5, 4, 3]（从索引 5 反向取到索引 3）
```

```python
res=all([v>0 for v in l])
res2=any([v>0 for v in l])

l2=[l**2 for i in range(5)]
a=max([v*v for v in l])#从列表到列表
l_only_even=[v for v in l if v%2==0]#更复杂
l_only

for index,v in enumerate(l):
	print("no."index,"",v)

for i in range(4,10,2):
	print(i)

# A と B から 1 つずつ要素を取得し a および b に格納する
for a, b in zip(A, B):#联合调用
    print(a, b)
     
```

```python
print("{} {}".format(a+b+c,s))
print(*s)#打印单行list或者字符串
```

```python
# 入力
N, K = map(int, input().split())
S = input().split()
 
# K文字以上の単語を抽出
result = [word for word in S if len(word) >= K]
 
# 空白区切りで出力
print(*result)
```

```python
n=2
m=3
a=[[0 for _ in range(n)] for _ in range(m)]

a=[[] for i in range(n)]#空

n, m = map(int, input().split())

ablist = [list(map(int, input().split())) for i in range(m)]

res = [["-"] * n for _ in range(n)]  # 使用res和后面一致

for a, b in ablist:
    a -= 1
    b -= 1
    res[a][b] = "o"
    res[b][a] = "x"

for row in res:
    print(" ".join(row))

```

```python
#recursion
def factorial(n):
	if n==0 or n==1:
		return 1
	s=factorial(n)
	return s*n
	
```

```python

n = int(input())
print(n)

```

---

```python
a, b, c = map(int, input().split())
print(a, b, c)
print(*[a, b, c])

```

---

```python

lst = list(map(int, input().split()))
print(lst)     # → [1, 2, 3]
print(*lst)    # → 1 2 3
```

---

```python
s = input().strip()
print(s)

```

---

```python
H, W = map(int, input().split())
grid = [list(map(int, input().split())) for _ in range(H)]
for row in grid:
    print(*row)

```

---

## 6. 高速入出力（オプション）

大量のデータを扱う場合は

```python

import sys
input = sys.stdin.readline
```

として`input()`を高速化できます。 

```python
import sys
input = sys.stdin.readline

n = int(input())
arr = list(map(int, input().split()))
print(sum(arr))

```
