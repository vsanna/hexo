title: "Facebook Hacker Cup 2015 Qualification Round"
date: 2016-01-04 20:43:22
tags:
- esplo77
- HackerCup
- C++11
---

[問題文](https://www.facebook.com/hackercup/problem/582062045257424/)
[解説 & Input/Outputデータ](https://www.facebook.com/notes/facebook-hacker-cup/hacker-cup-2015-qualification-round-solutions/1043281905687710/)

## 15: Cooking the Books
基本的な全探索が出来るかを見る問題。

ある2つの数を入れ替えて、最小・最大の数値が何になるかを求める問題。
先頭が0になるケースを除外し、全パターンを試せば良い。
$N$がintに収まるので数値型に入れたくなるが、桁数は変わらないので、もちろん文字列で扱ったほうが楽。

```C++
int main() {
	int T;
    cin >> T;
    rep(_test, T) {
        string str;
        cin >> str;

        int n = str.size();

        string mn = str, mx = str;
        rep(i, n) {
            REP(j, i+1, n) {
                if(str[j] == '0' && i == 0)
                    continue;

                swap(str[i], str[j]);
                chmin(mn, str);
                chmax(mx, str);
                swap(str[i], str[j]);
            }
        }

        cout << "Case #" << _test+1 << ": ";
        cout << mn << " " << mx << endl;
    }
	
    return 0;
}
```

## 30: New Year's Resolution
組み合わせの全探索が出来るかを見る問題っぽい。

いかにもDPっぽいが、値を持つのが3種類あるので、TLEしないDP解は思いつかなかった。
代わりに、各種類で目的の数字を作れる組み合わせを列挙した。

$N=20$なので、全組み合わせが列挙できる。
それぞれの種類で有効な組み合わせを全部列挙して、3種類全てである組み合わせが存在すれば"yes"、無ければ"no"を出力すれば良い。

3種類を上手く扱う方法が色々ありそうだが、以下では関数を作ってフィルタリングをするように繰り返し処理をしてみた。

```C++
int N;
vi solve(const vi& S, const vi& data, int target) {
    vi ans;
    tr(it, S) {
        int now = 0;
        rep(i, N) {
            if( ((1 << i) & (*it)) != 0) {
                now += data[i];
            }
        }
        if(now == target)
            ans.push_back(*it);
    }
    return ans;
}

int main() {
	int T;
    cin >> T;
    rep(_t, T) {

        vi target(3);
        rep(i,3)
            cin >> target[i];

        cin >> N;
        vector<vi> G(3, vi(N));
        rep(i, N)
            rep(j, 3)
                cin >> G[j][i];

        vi ans(1<<N);
        rep(i, (1<<N))
            ans[i] = i;

        rep(i, 3) {
            vi s1 = solve(ans, G[i], target[i]);
            ans = s1;
        }

        cout << "Case #" << _t + 1 << ": ";
        if(ans.empty())
            cout << "no";
        else
            cout << "yes";
        cout << endl;

    }
	
    return 0;
}
```


## 55: Laser Maze
ちょっと変わったBFSが書けるかを見る問題？

スタートからゴールの最短距離を見つけるので、BFSすれば良い。
ただし、回転するレーザーがあるため、各マスはステップ数に応じた状態を持つ必要がある。
レーザーは各ステップで90°回転するので、状態は4つ持てば十分。
どのステップで何処が通れなくなるかを事前に保持し、BFSすれば良い。

以下では、「そのマスへの最短距離」「壁、レーザー柱、レーザーが通る」という状態両方を持つdp配列を作った。
結構な実装量になってしまったが、シンプルに書けるのだろうか。

```C++
const int MAX_M = 102;
const int MAX_N = 102;

// dp[r][c][i] := 座標r,cにi回目（mod 4）で到達できる最短ステップ
int dp[MAX_M][MAX_N][4];

int dx[] = {1, 0, -1, 0};
int dy[] = {0, 1, 0, -1};

// coordinate to int
int p2i(const pii &co) {
    return co.first * MAX_N + co.second;
}

pii i2p(int t) {
    return mp(t / MAX_N, t % MAX_N);
}

string field[MAX_M];

bool isBlock(int r, int c) {
    switch (field[r][c]) {
        case '#':
        case '^':
        case '>':
        case 'v':
        case '<':
            return true;
        default:
            return false;
    }
}

int main() {

    int T;
    cin >> T;
    rep(_t, T) {
        int M, N;
        cin >> M >> N;

        memset(dp, 0, sizeof(dp));
        rep(r, M) rep(c, N) rep(i, 4) dp[r][c][i] = INF;
        pii start, goal;

        rep(r, M)
            cin >> field[r];

        rep(r, M) {
            rep(c, N) {
                if (!isBlock(r, c)) {
                    if (field[r][c] == '.')
                        continue;
                    else if (field[r][c] == 'S')
                        start = mp(r, c);
                    else if (field[r][c] == 'G')
                        goal = mp(r, c);
                }
                else {  // 通れない
                    rep(i, 4) dp[r][c][i] = -1;

                    if (field[r][c] == '#')
                        continue;

                    // 発射台を探し、向きと射線を記録
                    int t = 0;
                    if (field[r][c] == '^')
                        t = 0;
                    else if (field[r][c] == '>')
                        t = 3;
                    else if (field[r][c] == 'v')
                        t = 2;
                    else if (field[r][c] == '<')
                        t = 1;

                    int t1 = t, t2 = (t + 1) % 4, t3 = (t + 2) % 4, t4 = (t + 3) % 4;
                    for (int i = r - 1; i >= 0; --i) {
                        if (isBlock(i, c)) break;
                        dp[i][c][t1] = -1;
                    }
                    REP(i, c + 1, N) {
                        if (isBlock(r, i)) break;
                        dp[r][i][t2] = -1;
                    }
                    REP(i, r + 1, M) {
                        if (isBlock(i, c)) break;
                        dp[i][c][t3] = -1;
                    }
                    for (int i = c - 1; i >= 0; --i) {
                        if (isBlock(r, i)) break;
                        dp[r][i][t4] = -1;
                    }
                }
            }
        }


        queue<pii> que;
        que.push(mp(p2i(start), 0));

        int ans = -1;
        while (!que.empty()) {
            pii _p = que.front();
            que.pop();
            pii now = i2p(_p.first);
            int count = _p.second;

            dp[now.first][now.second][count % 4] = count;

            // ゴール判定
            if (mp(now.first, now.second) == goal) {
                ans = count;
                break;
            }

            // 異動先があるか？
            rep(i, 4) {
                int nr = now.first + dy[i];
                int nc = now.second + dx[i];

                // 画面外
                if (nr < 0 || M <= nr || nc < 0 || N <= nc)
                    continue;
                // 通れない
                if (dp[nr][nc][(count + 1) % 4] == -1)
                    continue;
                // 最小値を更新できるなら
                if (chmin(dp[nr][nc][(count + 1) % 4], count + 1))
                    que.push(mp(p2i(mp(nr, nc)), count + 1));
            }
        }

        cout << "Case #" << _t + 1 << ": ";
        if (ans == -1)
            cout << "impossible";
        else
            cout << ans;
        cout << endl;
    }

    return 0;
}
```


## まとめ
Qualificationは1問でも解ければ良いが、コンテストが終わるまで結果が分からない。
思わぬバグがあるかもしれないので、保険も兼ねて全部投げたほうが良さそう。

