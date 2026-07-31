---
title: "new抛出的异常"
publish: false
tags: ["C++"]
---
# new抛出的异常

メモリ領域を確保できなかった場合に std::bad_alloc 例外を投げます。配列
の new 演算子では、動的確保した配列の長さが初期化リストの長さより短い場合に
std::bad_array_new_length 例外を投げます。
