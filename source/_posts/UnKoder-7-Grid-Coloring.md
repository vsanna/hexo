title: "UnKoder #7 Grid Coloring"
date: 2015-06-08 22:14:51
tags:
- esplo77
- HackerRank
- UnKoder
- C++11
---

# Grid Coloring (problem 3)

## 問題

[https://www.hackerrank.com/contests/unkoder-07/challenges/grid-coloring](問題文)

## 考え方

赤いマスがどのマスになるかを考えて…。
とすると爆死コース。

あるマスの色を適当に決めて、そこから隣接するマスを違う色で塗る。
「全マスを塗る」「2色しかない」ということは、'#'で囲まれた領域を塗るパターンは2つしかないということ。
そのため、'#'で囲まれた領域ごとに適当に色A、色Bで塗る試行をし、マスが多い方を赤とすればよい。

## コード

```C++
const int MAX = 51;
char a[MAX][MAX];
int n, m;

int dy[] = {0, 1, 0, -1};
int dx[] = {1, 0, -1, 0};

int A, B;
int dfs(int y, int x, bool col){
    if(a[y][x] == '#')
        return 0;

    a[y][x] = '#';
    if(col) A++;
    else B++;

    rep(i,4) {
        int ty = y + dy[i];
        int tx = x + dx[i];
        if(0 <= ty && ty < n && 0 <= tx && tx < m) {
            dfs(ty, tx, !col);
        }
    }

    return max(A,B);
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    cin >> n >> m;

    rep(i,n)
        rep(j,m)
            cin >> a[i][j];

    int res = 0;
    rep(i,n) {
        rep(j, m) {
            A = B = 0;
            res += dfs(i, j, true);
        }
    }

    cout << res << endl;
    return 0;
}
```
