title: "Facebook Hacker Cup 2015 Round 1"
date: 2016-01-06 00:31:00
tags:
- esplo77
- HackerCup
- C++11
---

[問題文](https://www.facebook.com/hackercup/problem/582396081891255/)
[解説 & Input/Outputデータ](https://www.facebook.com/notes/facebook-hacker-cup/hacker-cup-2015-round-1-solutions/1047761065239794)
[オンラインジャッジ](http://codeforces.com/gym/100579)

## 10: Homework
エラトステネスの篩が実装できるかを見ているようだ。

事前に篩の変形版を作っておき、それを使い回せばよい。
篩の変形を作るのはおおよそ$O(nloglogn)$、クエリ処理は$O(n)$となる。

作りたい篩は、各数の素数の個数を記録するもの。
そのため、まず全て0で初期化した長さ$n$の篩を作る。
2～nまでループし、現在見ている値が素数（＝篩の値が0）であれば、その倍数の値をインクリメントしてゆけば良い。

このような処理では、いつもの篩とは異なり、ちゃんと$n$までループを回さないと素数か判定出来ないため注意。

```C++
const int MAX_N = (int)(1e7) + 10;

int main() {
    vi sieve(MAX_N, 0);
    for(int i=2; i <= MAX_N; ++i)
        if(sieve[i] == 0)
            for (int j = i; j <= MAX_N; j += i)
                sieve[j]++;

    int T;
    cin >> T;
    rep(_t, T) {
        int ans = 0;

        int A, B, K;
        cin >> A >> B >> K;
        REP(i, A, B+1)
            if(sieve[i] == K)
                ans++;

        cout << "Case #" << _t+1 << ": ";
        cout << ans << endl;
    }

    return 0;
}
```


## 25: Autocomplete
木構造が作れるかを見ているような。

各クエリでルートから木を辿る。
木のノードは、文字とその子ノードを持つ。
与えられた文字列を1文字ずつループし、各文字で以下の選択を行う。

- その文字が子要素に無い場合、ノードを追加する
  - 初回だけ、答えに加算する
- 子要素に持つ場合、その子に潜る

順番にこのような処理を行うことで、深さ1には1文字目の文字たち、深さ2には2文字目……、という木が出来上がる。

文字の長さは全て足しても$10^7$のようなので、ノード数は高々$10^7$。
子供をhash_mapで持てば速いはず。

なお、Nodeはグローバルに持たないとセグフォするので注意。

```C++

struct Node {
    int val;
    map<int, Node> children;
    Node(int v): val(v) {}
    // avoid g++ problem
    Node() {}
};

string str;
Node root(-1);
int main() {
	int T;
    cin >> T;
    rep(_t, T) {
        int N;
        cin >> N;

        int ans = 0;
        rep(i, N) {
            Node* now = &root;

            cin >> str;
            bool isNotFound = false;
            rep(j, str.size()) {
                char c = str[j] - 'a';
                if(now->children.find(c) == now->children.end()) {
                    if(!isNotFound)
                        ans += min(j + 1, (int) str.size());
                    isNotFound = true;
                    now->children.insert(mp(c, Node(c)));
                }
                now = &now->children[c];
            }
            if(!isNotFound)
                ans += str.size();
        }

        cout << "Case #" << _t+1 << ": ";
        cout << ans << endl;
    }
    return 0;
}
```

## 25: Winning at Sports
DPが書けるかどうかを見ているに違いない。

dp[i][j] := iを自分の勝利数、jを相手の勝利数とした時のパターン数
とし、順番にそれぞれが勝つパターンを辿れば良い。

stress-freeの時は$i>j$（ただし$i=0,j=0$の時を除く）が常に成り立ち、
stressfulの時は$i<=j$が成り立つことに留意してループする。

今回は、ループで辿る位置を制限し、雑に配るDPを行った。

```C++
int main() {
    int T;
    cin >> T;
    rep(_t, T) {
        int my, you;
        char c;
        cin >> my >> c >> you;

        int max_n = max(my, you) + 3;

        int ans1, ans2;
        {
            vector<vll> dp(max_n, vll(max_n, 0));
            dp[1][0] = 1;

            // i > j
            REP(i, 1, max_n-2) {
                rep(j, i) {
                    if (dp[i][j] == 0) continue;

                    dp[i + 1][j] += dp[i][j];
                    dp[i+1][j] %= MOD;
                    dp[i][j + 1] += dp[i][j];
                    dp[i][j+1] %= MOD;
                }
            }
            ans1 = dp[my][you];
        }

        {
            vector<vll> dp(max_n, vll(max_n, 0));
            dp[0][0] = 1;

            // i <= j
            rep(j, max_n-2) {
                rep(i, max_n-2) {
                    if(j != you && i > j) continue;

                    dp[i + 1][j] += dp[i][j];
                    dp[i+1][j] %= MOD;
                    dp[i][j + 1] += dp[i][j];
                    dp[i][j+1] %= MOD;
                }
            }
            ans2 = dp[my][you];
        }

        cout << "Case #" << _t+1 << ": ";
        cout << ans1 << " " << ans2 << endl;
    }
    return 0;
}
```

## 40: Corporate Gifting
木とDPのコンビネーションが出来るかどうかを見ているのかも。

各人員をノードとして、ルートがCEOとなる木を作る。
ノードvに対し、自分のギフト番号がiの時、以下のdp表を定義する。
dp[v][i] = vがルートの部分木で必要な最小コスト

直属の子ノードはiを使えない事に注意して探索する。
これを葉から順に埋めてゆき、最終的にdp[0][i]のなかで最小のものが答えになる。

公式の解説を見ると、ギフト番号の最大値は$log(n)$になるので、$O(Nlog^2N)$で求まる。

なお、以下は再帰で求めているので、通常のg++の設定ではStack Overflowする可能性が高い。
コドフォの設定と同様に、
-Wl,--stack=268435456
を付けてごまかした。

```C++
const int MAX_N = 200010;
vi tree[MAX_N];
const int LOG_N = log(MAX_N) + 2;
int dp[MAX_N][LOG_N];

int rec(int v, int c) {
    int res = c;
    if(dp[v][c] != -1)
        return dp[v][c];

    tr(it, tree[v]) {
        int mm = INF;
        REP(i, 1, LOG_N) {
            if(i==c) continue;
            chmin(mm, rec(*it, i));
        }
        res += mm;
    }
    return (dp[v][c] = res);
}

int main() {
    int T;
    cin >> T;
    rep(_t, T) {
        rep(i, MAX_N)
            tree[i].clear();
        memset(dp, -1, sizeof(dp));

        int N;
        cin >> N;

        rep(i, N) {
            int boss;
            cin >> boss;
            if(boss == 0) continue;
            tree[boss-1].push_back(i);
        }

        int ans = INF;
        REP(i, 1, LOG_N)
            chmin(ans, rec(0,i));

        cout << "Case #" << _t+1 << ": ";
        cout << ans << endl;
    }
    return 0;
}
```


## まとめ
この年のRound 1は全完しないと次に進めなかったようだ。
世知辛すぎる。