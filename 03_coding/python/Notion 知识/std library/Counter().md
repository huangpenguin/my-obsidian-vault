---
title: "Counter()"
publish: false
tags: ["Python"]
---
# Counter()

```python
from collections import Counter
breakfast=['spam','spam','spam','egg']
breakfast_counter=Counter(breakfast)
breakfast_counter.most_common(1)#降顺

lunch=['eggs','eggs','bacon']
lunch_counter=Counter(lunch)

breakfast_counter+lunch_counter
breakfast_counter-lunch_counter#早餐吃了午餐没吃

breakfast_counter&lunch_counter#交集合
breakfast_counter|lunch_counter#并集合
```
