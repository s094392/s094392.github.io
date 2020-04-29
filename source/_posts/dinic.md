---
title: Dinic 模板
date: 2018-03-30 03:37:43
tags:
categories: [Algorithm]
---
上學期寫的
要整理電腦所以乾脆備份上來
<!--more-->

```cpp
// checked POJ-1273
# define INF 1023456789

bool bfs(int s, int t, vector<vector<int>>& adj, vector<bool>& visited,
         vector<int>& d) {
    queue<int> Q;
    Q.push(s);
    visited[s] = 1;
    while (!Q.empty()) {
        int now = Q.front();
        for (int i = 0; i < adj[now].size(); i++) {
            if (adj[now][i] > 0 && !visited[i]) {
                d[i] = d[now] + 1;
                visited[i] = 1;
                if (i == t) return 1;
                Q.push(i);
            }
        }
        Q.pop();
    }
    return 0;
}

int dfs(int now, int maxf, int s, int t, vector<vector<int>>& adj,
        vector<bool>& visited, vector<int>& d) {
    if (now == t) return maxf;
    if (visited[now]) return 0;
    visited[now] = 1;
    for (int i = 0; i < adj[now].size(); i++) {
        int tmp = adj[now][i];
        if (tmp > 0 && d[i] == d[now] + 1) {
            int f = dfs(i, min(tmp, maxf), s, t, adj, visited, d);
            if (f) {
                adj[now][i] -= f;
                adj[i][now] += f;
                return f;
            }
        }
    }
    return 0;
}

int solve(int n, int m) {
    /*
     * m dots
     * n edges
     * s = source
     * t = sink
     */
    int a, b, c, s, t, flow;
    s = 1;
    t = m;
    vector<vector<int>> adj(m + 2, vector<int>(m + 2, 0));
    vector<bool> visited(m + 2, 0);
    vector<int> d(m + 2, 0);  // depth
    for (int i = 0; i < n; i++) {
        cin >> a >> b >> c;
        adj[a][b] += c; // reminds of repeated edges!
    }
    flow = 0;
    while (bfs(s, t, adj, visited, d)) {
        while (1) {
            visited.clear();
            visited.resize(m + 2, 0);
            int f = dfs(s, INF, s, t, adj, visited, d);
            if (!f) break;
            flow += f;
        }
        visited.clear();
        visited.resize(m + 2, 0);
        d.clear();
        d.resize(m + 2, 0);
    }
    cout << flow << "\n";
}

int main() {
    int n, m;
    while (cin >> n >> m) solve(n, m);
}
```
