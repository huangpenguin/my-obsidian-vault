---
title: "ATM类控制取钱"
publish: false
tags: ["Python"]
---
# ATM类控制取钱

```python
class Bank():
    crisis = False
    def create_atm(self):
        while not self.crisis:
            yield "$100"

hsbc = Bank()
corner_street_atm = hsbc.create_atm()
print(next(corner_street_atm)) 
print(next(corner_street_atm)) 
print([next(corner_street_atm) for _ in range(5)]) 
hsbc.crisis = True#危机开始，while循环结束
print(next(corner_street_atm)) 
```

- **设置`hsbc.crisis`为`True`并再次调用`next()`**:
    
    当`hsbc.crisis`设置为`True`后，`while not self.crisis`条件不再为真，生成器不再生成值。因此，调用`next(corner_street_atm)`会引发`StopIteration`异常。
    

```python
  def create_atm(self):
        while not self.crisis:
            yield "$100"
```

所以上面这一坨是生成器部分，每次调用都会进行一次while的判断
