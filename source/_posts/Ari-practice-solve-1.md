title: "蟻本練習問題を解いた #1"
date: 2015-08-10 00:10:43
tags:
- esplo77
- C++
- 蟻本
---

# 回答たち
POJではC++11が使えない。
また、bits/stdc++.hが無いので置き換えないといけなかった。
そのうえ、OnlineJudgeHelperが何故か例外を吐くので手動コピーした。
これは面倒。

## POJ1979 - Red and Black
dfsするだけ。
迷路はboolの配列で作った。

```C++
const int MAX = 22;
bool maze[MAX][MAX];
bool vis[MAX][MAX];

int dx[] = {1,0,-1,0};
int dy[] = {0,1,0,-1};

int dfs(int r, int c) {
    if(vis[r][c])
        return 0;
    vis[r][c] = true;

    int t = 1;
    rep(i,4) {
        int nr = r + dy[i];
        int nc = c + dx[i];
        if(maze[nr][nc])
            t += dfs(nr, nc);
    }
    return t;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    cin.tie(0);
    ios_base::sync_with_stdio(false);

    int W, H;
    while(cin >> W >> H, W || H) {

        memset(maze, 0, sizeof(maze));
        memset(vis, 0, sizeof(vis));

        int myr = 0, myc = 0;

        rep(i, H) {
            string line;
            cin >> line;
            rep(j, W) {
                if (line[j] == '.')
                    maze[i][j] = 1;
                else if (line[j] == '@')
                    myr = i, myc = j;
            }
        }

        cout << dfs(myr, myc) << endl;
    }

    return 0;
}
```


## POJ3009 - Curling 2.0
最初幅優先で解いて提出したが、何故かWA。
仕方ないので、DFSで書きなおした。
全探索をする場合、DFSでも幅優先と同じことができることがある。
↓に詳しく解説されている。
http://www.deqnotes.net/acmicpc/3009/

```C++
int dx[] = {1,0,-1,0};
int dy[] = {0,-1,0,1};

int H, W;
const int MAX=22;
bool block[MAX][MAX];
int gr;
int gc;
int ans;

void dfs(int r, int c, int depth) {
    if(depth >= ans)
        return ;

    rep(i,4) {
        int nr = r;
        int nc = c;
        for(int count=0; ; count++) {
            nr += dy[i];
            nc += dx[i];

            if(nr < 0 || H <= nr || nc < 0 || W <= nc)
                break;

            if(count==0)
                if(block[nr][nc])
                    break;

            if(nr==gr && nc==gc) {
                ans = depth;
                return ;
            }
            else if (block[nr][nc]) {
                block[nr][nc] = false;
                dfs(nr - dy[i], nc - dx[i], depth+1);
                block[nr][nc] = true;
                break;
            }
        }
    }
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    while(cin>>W>>H, W||H) {
        memset(block, 0, sizeof(block));

        ans = 11;
        int sr = 0, sc = 0;
        gr = 0, gc = 0;

        rep(i, H) {
            rep(j, W) {
                int t;
                cin >> t;
                if (t == 1)
                    block[i][j] = true;
                else if (t == 2)
                    sr = i, sc = j;
                else if (t == 3)
                    gr = i, gc = j;
            }
        }

        dfs(sr, sc, 1);

        if(ans>10)
            cout << -1 << endl;
        else
            cout << ans << endl;
    }

    return 0;
}
```


## AOJ0118 - Property Distribution
蟻本の水たまりと同じ方法で解ける。
DFSの仕組みがイメージできないといけないので、初心者にオススメっぽい。

```C++
int H, W;
const int MAX=102;
string field[MAX];
bool vis[MAX][MAX];

int dx[] = {1,0,-1,0};
int dy[] = {0,1,0,-1};

const char DUMMY = '-';

int dfs(int r, int c) {

    if(vis[r][c])
        return 0;

    char floor = field[r][c];
    field[r][c] = DUMMY;
    vis[r][c] = true;

    rep(i,4) {
        int nr = r + dy[i];
        int nc = c + dx[i];

        if(nr < 0 || H <= nr || nc < 0 || W <= nc)
            continue;

        if(floor == field[nr][nc])
            dfs(nr, nc);
    }

    return 1;
}


int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    while(cin>>H>>W, (H||W)) {
        memset(vis,0,sizeof(vis));

        rep(i,H)
            cin >> field[i];

        int res = 0;
        rep(i,H) {
            rep(j,W) {
                if(field[i][j] != DUMMY)
                    res += dfs(i, j);
            }
        }

        cout << res << endl;
    }

    return 0;
}
```

## AOJ0033 - Ball
制約がかなり緩いので何でも解ける。
とりあえず、簡単そうなビット全探索をした。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;

    while(N--) {
        int a[10];
        rep(i, 10)
            cin >> a[i];

        bool probEnd = false;
        rep(i, 2<<10) {
            int left = 0, right = 0;
            bool failed = false;

            rep(j,10) {
                bool isRight = ( (i>>j) & 0x1 );
                if(isRight) {
                    if(right > a[j]) {
                        failed = true;
                        break;
                    }
                    right = a[j];
                }
                else {
                    if(left > a[j]) {
                        failed = true;
                        break;
                    }
                    left = a[j];
                }
            }

            if(!failed) {
                cout << "YES" << endl;
                probEnd = true;
                break;
            }
        }

        if(!probEnd)
            cout << "NO" << endl;
    }

    return 0;
}
```


## 感想
まだまだ簡単。
