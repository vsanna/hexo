title: "Ari-practice solve #2"
date: 2015-08-15 17:54:26
tags:
- esplo77
- C++
- 蟻本
---

# 蟻本練習問題を解いた (2)
探索&探索&探索。
dx, dyをテンプレートに加えたくなった。

ちなみに、x,yで2次元配列を扱うとa[x][y]とかで間違えそうなので、r,cを使っている。

## AOJ0558 - Cheese
シンプルな幅優先を繰り返すだけ。

```C++
int dx[] = {1,0,-1,0};
int dy[] = {0,1,0,-1};
 
 
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);
 
    int H,W,N;
    cin >> H >> W >> N;
 
    int a[H][W];
    memset(a, 0, sizeof(a));
    int vis[H][W];
    memset(vis, -1, sizeof(vis));
 
    int start = 0;
    rep(r,H) {
        string t;
        cin >> t;
        rep(c,W) {
            if(t[c] == 'S')
                start = r*W + c;
            else if(t[c] == 'X')
                a[r][c] = -1;
            else if(t[c] != '.')
                a[r][c] = t[c] - '0';
        }
    }
 
 
    int ans = 0;
 
    REP(target,1,N+1) {
        queue<int> que;
        que.push(start);
 
        memset(vis, -1, sizeof(vis));
        vis[start / W][start % W] = 0;
 
        while (!que.empty()) {
            int t = que.front(); que.pop();
            int r = t / W;
            int c = t % W;
 
            if (a[r][c] == target) {
                ans += vis[r][c];
                start = r * W + c;
                break;
            }
 
            rep(i, 4) {
                int nr = r + dy[i];
                int nc = c + dx[i];

                if (nr < 0 || H <= nr || nc < 0 || W <= nc
                    || a[nr][nc] == -1 || vis[nr][nc] != -1)
                    continue;

                vis[nr][nc] = vis[r][c] + 1;
                que.push(nr * W + nc);
            }
        }
    }

    cout << ans << endl;

    return 0;
}
```

## POJ3669 - Meteor Shower
同じくシンプルな幅優先。
距離を表す配列はINFで初期化しよう。

```C++
const int MAX = 305;
int field[MAX][MAX];
int vis[MAX][MAX];


int fdx[] = {1,0,-1,0};
int fdy[] = {0,1,0,-1};

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    rep(i,MAX)
        fill(field[i], field[i]+MAX, INF);
    rep(i,MAX)
        fill(vis[i], vis[i]+MAX, INF);

    int M;
    cin >> M;

    rep(query,M) {
        int x,y,t;
        cin >> x >> y >> t;
        field[y][x] = min(t, field[y][x]);

        rep(i,4) {
            int ny = y + fdy[i];
            int nx = x + fdx[i];

            if(ny < 0 || MAX <= ny || nx < 0 || MAX <= nx)
                continue;
            field[ny][nx] = min(t, field[ny][nx]);
        }
    }

    queue<int> que;
    que.push(0);
    vis[0][0] = 0;

    while(!que.empty()) {
        int t = que.front(); que.pop();
        int y = t / MAX;
        int x = t % MAX;
        int time = vis[y][x];

        if(field[y][x] == INF) {
            cout << time << endl;
            return 0;
        }

        rep(i,4) {
            int ny = y + fdy[i];
            int nx = x + fdx[i];

            if(ny < 0 || MAX <= ny || nx < 0 || MAX <= nx)
                continue;
            if(vis[ny][nx] != INF)
                continue;
            if(field[ny][nx] <= time+1)
                continue;

            vis[ny][nx] = time+1;
            que.push(ny*MAX + nx);
        }
    }

    cout << -1 << endl;

    return 0;
}
```

## AOJ0121 - Seven Puzzle
盤面の状態を渡す系幅優先。
この問題は逆から全状態の距離を求める工夫が必要。

```C++
int dr[] = {0, 1, 0, -1};
int dc[] = {1, 0, -1, 0};

map<string, int> ans;

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    queue< pair<string, int> > que;
    que.push( mp("01234567", 0) );

    while (!que.empty()) {
        pair<string, int> t = que.front(); que.pop();
        int zero = t.first.find_first_of('0');

        ans[t.first] = t.second;

        rep(i, 4) {
            int nr = zero / 4 + dr[i];
            int nc = zero % 4 + dc[i];

            if (nr < 0 || 2 <= nr || nc < 0 || 4 <= nc)
                continue;

            swap(t.first[nr * 4 + nc], t.first[zero]);
            if( ans.find(t.first) == ans.end() )
                que.push(mp(t.first, t.second + 1));
            swap(t.first[nr * 4 + nc], t.first[zero]);
        }
    }

    int a[8];
    while (cin >> a[0]) {
        rep(i,7) cin >> a[i+1];
        stringstream ss;
        rep(i,8) ss << a[i];

        cout << ans[ss.str()] << endl;
    }

    return 0;
}
```

## 感想
すごく探索した。
