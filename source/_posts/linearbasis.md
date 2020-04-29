---
title: Linear basis for xor
date: 2018-01-16 23:22:49
tags: [Math] 
categories: [Algorithm]
---
練習的時候有幾次遇到 xor 的題目
自己怎想都想不到
然後去查大陸人的題解
大概都是說：看到 xor 就想到線性基阿，看到位元運算就想到線性基之類的東西
結果拿線性基一下去 google 才發現這就是線性代數裡面的 basis
<!--more-->

basis 就是組成(span)一個向量空間(vector space)的基本向量
舉個簡單的例子
$(1, 0), (0, 1)$ 就是一個 $R^2$ 的 vector space 的兩個 basis
至於這東西跟 xor 有什麼關係呢？
主要是跟 xor 的性質有關

我們先看 vector space 的八個定理（出自[維基百科](https://zh.wikipedia.org/wiki/%E5%90%91%E9%87%8F%E7%A9%BA%E9%97%B4)）

| 公理       | 	說明       |
| ------------- | ------------- |
|向量加法的結合律	|\\[u + (v + w) = (u + v) + w\\]|
|向量加法的交換律	|\\[u + v = v + u\\]|
|向量加法的單位元素	|\\[存在一個叫做零向量的元素0 ∈ V，使得對任意u ∈ V都滿足u + 0 = u\\]|
|向量加法的反元素	|\\[對任意v ∈ V都存在其反元素−v ∈ V使得v + (−v) = 0\\]|
|純量乘法與純量的體乘法相容	|\\[a(bv) = (ab)v\\]|
|純量乘法的單位元素	|\\[體F存在乘法單位元素1滿足1v = v\\]|
|純量乘法對向量加法的分配律	|\\[a(u + v) = au + av\\]|
|純量乘法對體加法的分配律	|\\[(a + b)v = av + bv\\]|

如果我們定義一個 space 是由 n 個二進位的數字互相 xor 可以生出來的數組成的
不難發現這個 space 正好滿足這八個公理，只要把加法換成 xor 就好了
所以呢，我們就可以維護一組 basis  來表示某數組可以 xor 出的所有數

方法如下：
一個一個數字來處理，如果當前這個數已經可以用目前的 bases span 出來，就跳過，否則就把它加進 bases 裡面

實做的方法就是把每個數用二進位表示成一個向量，然後維護一個 upper triangular 的 bases 矩陣，當拿到一個 n 位的數 a（以二進位來看），就先看看 bases 矩陣裡面有沒有 n 位數的 basis ，沒有的話就把它放進 bases 裡面，有的話就把 a ^= 那個 basis，如此一來 a 就會降位，然後一直重複降位直到 a 等於 0 或是bases 矩陣裡面有沒有 n 位數的 basis
```cpp
// add tmp
for (int j = n; j > 0; j--) {
    if (tmp >> (j - 1) & 1) {
        if (V[j])
            tmp ^= V[j];
        else {
            V[j] = tmp;
            break;
        }
    }
}
```
`V[1]` 放的是二進位是 `1` 位數的數，`V[n]`放的是二進位是 `n` 位數的數
首先從最高位開始找 tmp 的第一個 1，然後檢查 `V[i]` 有沒有東西


有一些時候會需要在 space 裡面找出一些符合題目要求的數（比如說可以 xor 出最大的數），這時候可以消去的時候順便把給個位數整理，讓每一個位數的 1 都只存在一個 basis 裡面
因此每次找到一個 basis 的時候，先把剛找到的那個後面的 1 都先儘量消掉，接著在區消掉前面比它位數多的 basis 的 1
```cpp
// add tmp into vector space
for (int j = n; j > 0; j--) {
    if (tmp >> (j - 1) & 1) {
        if (V[j])
            tmp ^= V[j];
        else {
            V[j] = tmp;
            for (int k = j - 1; k >= 0; k--)
	            if (V[k] && V[j] >> (k - 1) & 1) V[j] ^= V[k];
            for (int k = j + 1; k <= n; k++)
	            if (V[k] >> (j - 1) & 1) V[k] ^= V[j];
            break;
        }
    }
}
```
看一下實際的例子，以下模擬出自：[线性基学习笔记 Sengxian's Blog](https://blog.sengxian.com/algorithms/linear-basis)

> 我們來模擬下這個過程，$n = 5, a = \{7, 1, 4, 3, 5\}$。一開始矩陣是這樣的：
> 
> \begin{bmatrix} 0 & 0 & 0\\\ 0 & 0 & 0\\\ 0 & 0 & 0 \end{bmatrix}
> 
> ​​  加入 $7 = (111)_2$，矩陣變為：
> 
> \begin{bmatrix} 1 & 1 & 1\\\ 0 & 0 & 0\\\ 0 & 0 & 0 \end{bmatrix}
> 
> 加入 $1 = (001)_2$​​ ，添加到最後一行，同時為了維護對角矩陣，消去第一行的最低位，矩陣變為：
> 
> \begin{bmatrix} 1 & 1 & 0\\\ 0 & 0 & 0\\\ 0 & 0 & 1 \end{bmatrix}
> 
> 加入 $4 = (100)_2$ ，由於第一行已經有數了，它被 xor 為 $(010)_2$ ​​
> ，加入第二行，同時為了維護對角矩陣，消去第一行的第二位，矩陣變為：
> 
> \begin{bmatrix} 1 & 0 & 0\\\ 0 & 1 & 0\\\ 0 & 0 & 1 \end{bmatrix}

如此一來我們就得到一個上三角的矩陣
那麼要怎麼判端某個數是不是存在這個 space 呢？
作法就是從該數的最高位開始 xor 對應的 basis（如 1001 就先 `^= V[4]`），並重複做下去，如果最後變成 0 就代表他在這個 space 裡面

有關於 vector space 在 xor 的應用大概就是這樣
之所以會特別寫一篇筆記是因為競程二期末考得時候有一題就是出這個，而且真的很水，雖然考試當下我一眼就看出來是要用 basis 去算，但是那時候不知道怎麼快速的維護 basis  矩陣（以前有看過作法但是沒有理解），有點殘念所以回去之後跟教授要了題目跟測資自己寫自己 judge XD

至於題目的話是清大內部比賽的題目所以就不全部放上來了，這裡寫稍微編譯之後的題目敘述就好：

> 給一個 target ，以及 n 個數，然後求最少可以用第幾個數之前的數 xor 出 target 即可

我的方法是維護一個 bases 矩陣然後每次加完就 check 一下是不是已經可以 span 出 target 了

附上我寫的垃圾 code ：
```cpp
#include <bits/stdc++.h>
using namespace std;

#define ll long long
#define ui unsigned int
#define ull unsigned long long
#define INF 1023456789
#define jizz    \
    cin.tie(0); \
    ios_base::sync_with_stdio(0)

void solve() {
    ll n, t, tar = 0, tmp, ans = 0;
    cin >> n >> t;
    vector<ll> V(n + 2, 0);  // vector space
    string tmps;
    // target
    cin >> tmps;
    for (int i = 0; i < n; i++) {
        tar <<= 1;
        if (tmps[i] == '1') tar += 1;
    }
    for (int i = 1; i <= t; i++) {
        tmp = 0;
        cin >> tmps;
        if (!ans) {
            for (int j = 0; j < n; j++) {
                tmp <<= 1;
                if (tmps[j] == '1') tmp += 1;
            }
            // add tmp
            for (int j = n; j > 0; j--) {
                if (tmp >> (j - 1) & 1) {
                    if (V[j])
                        tmp ^= V[j];
                    else {
                        V[j] = tmp;
                        break;
                    }
                }
            }
            // check if the vector space can span the target
            for (int j = n; j > 0; j--) {
                if (tar >> (j - 1) & 1) {
                    if (V[j])
                        tar ^= V[j];
                    else
                        break;
                }
                if (j == 1) ans = (ll)i * (ll)i;
            }
        }
    }
    if (ans)
        cout << ans << "\n";
    else
        cout << -1 << "\n";
}

int main() {
    jizz;
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
}

```
在本機測最慢的冊資 0.82 s 應該是 OK 了
