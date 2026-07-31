---
title: "tuple"
publish: false
tags: ["Python"]
---
# tuple

最重要的一个特性就是不可变更性 immutable

```python
#生成tuple

t1='big foot',#tuple
t1='big foot'#string
t1=('big foot')#string

#最好!!!!!!!!!!!!!!使用括号并且尾随一个逗号，不容易引起歧义
t1=('big foot',)
type(t1)

#可以使用多重赋值
t1=('a','b','c')
a,b,c=t1

#可以进行变量置换
a='a'
b='b'
a,b=b,a

#可以从别的类型转换成tuple
t1= ['a'.'b']
tuple(t1)

#可以直接相连
('a',)+('b','c',)

#使用tuple遍历
words =('a','b','c')
for word in words

```

可以通过加号更新tuple的值，但是使用id（）函数来看，地址发生了改变也就是说生成了新的tuple

[import collections
import doctest

Card = collections.namedtuple('Card', ['rank', 'suit'])# 定义具名元组 Card
class FrenchDeck:
    """
    A deck of playing cards.
    
    >>> beer_card = Card('7', 'diamonds')
    >>> beer_card.rank
    '7'
    >>> beer_card.suit
    'diamonds1 '
    
    >>> deck = FrenchDeck()
    >>> len(deck)
    52
    >>> deck[0]
    Card(rank='2', suit='spades')
    >>> deck[:3]
    [Card(rank='2', suit='spades'), Card(rank='3', suit='spades'), Card(rank='4', suit='spades')]
    >>> deck[12::13]
    [Card(rank='A', suit='spades'), Card(rank='A', suit='diamonds'), Card(rank='A', suit='clubs'), Card(rank='A', suit='hearts')]
    
    >>> for card in reversed(deck): # doctest: +ELLIPSIS
    ...     print(card)
    Card(rank='A', suit='hearts')
    ...
    Card(rank='2', suit='spades')
    """
    
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
                                        for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]

if __name__ == "__main__":
    import doctest
    doctest.testmod()
](../../BOOK/fluent%20python/namedtuple%2022b64a37b2f0446684de8ad9aebee130.md)
