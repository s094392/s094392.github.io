---
title: FOJ 475. tetris
date: 2018-03-30 03:28:32
tags: [FOJ]
categories: [Algorithm]
---
競程作業，Assignment 6 (Week 5) - Tetris：[FOJ-475](https://oj.nctu.me/problems/475/)

模擬題，他要玩一個俄羅斯方塊遊戲
然後玩玩之後要問最高的方塊有多高
這題很直觀但很麻煩
<!--more-->
我的方法就是先用一個 unordered_map 去存每種方塊
然後維護一個每個 column 最高的位置
然後直接算出那個方塊會停在哪裡

而地圖就是開一個二維的 vector 然後發現有一個 row 全滿就 erase 掉
至於怎麼看有沒有全滿就是一開場就做一個全滿的 row 拿去 hash
之後就看 hash 出來是不是一樣就好了 XD
我也懶的研究 c++ 怎麼 hash 所以直接在開一個 map XDDD
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

int main()
{
    jizz;
    int p, w, ans = 0;
    cin >> p >> w;
    vector<vector<bool>> M(p * 4 + 10, vector<bool>(w, 0));
    vector<bool> tmpv(w, 1);
    vector<bool> em(w, 0);
    unordered_map<vector<bool>, bool> check;
    check[tmpv] = 1;
    char tmpc;
    string moves;
    unordered_map<char, pair<vector<int>, vector<int>>> cubes;
    cubes['L'] = make_pair(vector<int>({ 0, 3, 1 }), vector<int>({ 0, 0, 0 }));
    cubes['J'] = make_pair(vector<int>({ 0, 1, 3 }), vector<int>({ 0, 0, 0 }));
    cubes['O'] = make_pair(vector<int>({ 0, 2, 2 }), vector<int>({ 0, 0, 0 }));
    cubes['I'] = make_pair(vector<int>({ 0, 0, 4 }), vector<int>({ 0, 0, 0 }));
    cubes['Z'] = make_pair(vector<int>({ 1, 2, 1 }), vector<int>({ -1, 0, 0 }));
    cubes['S'] = make_pair(vector<int>({ 1, 2, 1 }), vector<int>({ 0, 0, -1 }));
    cubes['T'] = make_pair(vector<int>({ 1, 2, 1 }), vector<int>({ -1, 0, -1 }));
    vector<int> h(w + 5, 0);
    while (p--) {
        cin >> tmpc;
        cin >> moves;
        int place = 0;
        int width;
        auto nowc = cubes[tmpc];
        if (nowc.first[0] != 0)
            width = 3;
        else if (nowc.first[1] != 0)
            width = 2;
        else
            width = 1;
        for (auto& it : moves) {
            if (it == '>') {
                if (place != 0)
                    place--;
            } else {
                if (place + width != w)
                    place++;
            }
        }
        int maxh = -INF;
        for (int i = 0; i < width; i++)
            maxh = max(maxh, h[place + i] + nowc.second[width - i - 1]);
        for (int i = 0; i < width; i++) {
            if (nowc.first[2 - i]) {
                for (int j = maxh - nowc.second[2 - i]; j < maxh - nowc.second[2 - i] + nowc.first[2 - i]; j++) {
                    M[j][place + i] = 1;
                }
            }
        }
        for (int i = 0; i < 5; i++) {
            if (check.find(M[maxh + i]) != check.end()) {
                M.erase(M.begin() + maxh + i);
                i--;
                M.push_back(em);
            }
        }
        for (int i = 0; i < w; i++) {
            int tmph = 0;
            for (int j = 0; j < M.size(); j++) {
                if (M[j][i]) {
                    tmph = max(tmph, j + 1);
                }
            }
            h[i] = tmph;
        }
    }
    for (auto it : h)
        ans = max(it, ans);
    cout << ans << "\n";
}
```
