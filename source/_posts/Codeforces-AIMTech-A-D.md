title: "Codeforces AIM Tech Round Div.2 A-D"
date: 2016-02-06 19:15:00
tags:
- esplo77
- Codeforces
- C++11
---

## A. Save Luke
[問題文](http://codeforces.com/contest/624/problem/A)

### 考え方
左右の壁は逆向きなので、速度を足しあわせd以下になる時間を求めればよい。
物理の最初のテストで出そうな問題。

### コード

```C++
int main() {
	int d, L, v1, v2;
    cin >> d >> L >> v1 >> v2;

    cout << (double)(L-d) / (double)(v1+v2) << endl;
	
    return 0;
}
```

## B. Making a String
[問題文](http://codeforces.com/contest/624/problem/B)

### 考え方
大きい数から被っているかを考慮し、被っていれば1ずつ値を減少させてゆく。
最終的に0以外で被りが無くなれば、それが答えになる。

様々な解き方があると思うが、今回は条件を満たすまで貪欲にループした。

### コード

```C++
int main() {
	int n;
    cin >> n;
    map<int, int> nums;
    rep(i, n) {
        int a;
        cin >> a;
        nums[a]++;
    }

    while(true) {
        bool ok = true;
        tr(it, nums) {
            if(it->second > 1) {
                it->second--;
                nums[it->first - 1]++;
                ok = false;
            }
        }
        if(ok)
            break;
    }

    ll sum = 0;
    tr(it, nums) {
        if(it->first > 0)
            sum += it->first;
    }

    cout << sum << endl;
	
    return 0;
}
```

## C. Graph and String
[問題文](http://codeforces.com/contest/624/problem/C)

### 考え方
問題の条件をまとめると、以下の通り。

- 同じ文字同士でエッジがあっても構わない
- a, c間のみエッジがあってはいけない

このままでは扱いにくいため、否定をとって、エッジが無い部分の条件を考えてみる。
すると、「エッジが無いノード間は、aとc」という条件を考えれば良いことがわかる。
これにより、グラフの否定を取り（繋がっていない箇所を繋がっているとみなす）、そのグラフで二部グラフ判定（2色で色を塗る）を行えば良い。

なお、色を塗る際、孤立した点に色をつけない必要がある。
グラフの否定をとると、エッジの無い孤立した点が出てくるが、これは元のグラフですべてのノードと繋がっている。
そのため、bを与えるべき頂点という意味で色を付けていない。

これで、「繋がるべきでない頂点が正しいか」は判定でき、その際の色分けも判明したので、最後に「繋がっている頂点が正しいか」の判定を行えば良い。
この判定を忘れると、Test #26で落ちることになる（実体験）。

```text
7 13
1 2
2 3
1 3
4 5
5 6
4 6
2 5
2 7
3 7
7 4
7 6
7 1
7 5
```

付けた色を使って、-1 -> a、 0 -> b、 1 -> c と文字に対応付ければ答えになる。

### コード

```C++
const int MAX_V = 510;
int mat[MAX_V][MAX_V];
vi color(MAX_V, 0);
int n;

bool dfs(int v, int c) {
    color[v] = c;

    rep(i, n) {
        if(v != i && mat[v][i] == 1) {
            if (color[i] == c)
                return false;
            if(color[i] == 0 && !dfs(i, -c))
                return false;
        }
    }
    return true;
}

int main() {
    rep(i, MAX_V)
        fill(mat[i], mat[i] + MAX_V, 1);

	int m;
    cin >> n >> m;

    rep(i, m) {
        int u, v;
        cin >> u >> v;
        --u, --v;
        mat[u][v] = mat[v][u] = 0;
    }

    // check NOT connected edges
    rep(i, n) {
        if(color[i] == 0) {
            bool isIso = true;
            REP(j, i+1, n) {
                if(mat[i][j] == 1) {
                    isIso = false;
                    break;
                }
            }

            // 孤立している点は放置
            if(isIso)
                continue;

            if(!dfs(i, 1)) {
                cout << "No" << endl;
                return 0;
            }
        }
    }

    // check connected edges
    rep(i, n) {
        REP(j, i+1, n) {
            // つながっていて、色が反対
            if(mat[i][j] == 0 && color[i] != 0 && color[i] * -1 == color[j]) {
                cout << "No" << endl;
                return 0;
            }
        }
    }

    cout << "Yes" << endl;
    rep(i, n) {
        if(color[i] == -1)
            cout << 'c';
        else if(color[i] == 0)
            cout << 'b';
        else
            cout << 'a';
    }
    cout << endl;
	
    return 0;
}
```

## D. Array GCD
[問題文](http://codeforces.com/contest/624/problem/D)

### 考え方
まず、数列全体のgcdを取るということは、全要素が最初か最後の素因数のどれかで割り切れるということ。
取りうる値は、$a[0]-1$, $a[0]$, $a[0]+1$, $a[n-1]-1$, $a[n-1]$, $a[n-1]+1$の素因数のいずれか。
これは高々$O(log(max(a_i)))$で求まる。

次に、削除は1度しかできないため、部分文字列は{0: 削除していない, 1:削除中、2:削除完了}の3状態を持つことで、以下の漸化式がかける。

割り切れる場合（costは変更する必要があればb、なければ0）では、
$$
dp[i+1][0] = dp[i][0] + cost \\\
dp[i+1][2] = dp[i][1] + cost \\\
dp[i+1][2] = dp[i][2] + cost
$$

また、削除処理は割り切れるかどうかに関わらず行える。
$$
dp[i+1][1] = dp[i][0] + a \\\
dp[i+1][1] = dp[i][1] + a
$$

このようにして求めたDP表で、dp[n+1]の最小が答えになる。
なお、dp[n+1][0]が最小なら削除なし、
dp[n+1][1]が最小なら最後まで削除、
dp[n+1][2]が最小ならどこかを削除、となる。

なお、DP表は直前だけ記憶すれば良いので、使用メモリを大きく減らせるが、
制約が緩いのでわかりやすさを重視した。

### コード

```C++
// 1は除外する
set<int> enumPrimeFactors(int a) {
    set<int> ans;
    for (int i = 2; i * i <= a; i++) {
        while (a % i == 0) {
            ans.insert(i);
            a /= i;
        }
    }
    // のこり
    if(a != 1)
        ans.insert(a);

    return ans;
}

int main() {
    const ll INF = (1ll << 60);

    int n, a, b;
    cin >> n >> a >> b;
    vi nums(n);
    rep(i, n) cin >> nums[i];

    set<int> factors;
    // enumerate primes
    REP(i, -1, 1 + 1) {
        set<int> tmp = enumPrimeFactors(nums[0] + i);
        set<int> tmp2 = enumPrimeFactors(nums[n - 1] + i);
        factors.insert(all(tmp));
        factors.insert(all(tmp2));
    }

    ll ans = INF;

    // dp[i][j] := i番目までの数列を処理し、j={0: 削除していない, 1:削除中、2:削除完了}
    vector<vll> dp(n + 1, vll(3, INF));      // ループ内で確保するとTLEするため
    tr(fact, factors) {
        rep(i, n+1)
            fill(all(dp[i]), INF);
        // 初期状態、1つ目を削る状態
        dp[0][0] = 0;

        rep(i, n) {
            // change
            REP(j, -1, 1+1) {
                if( (nums[i]+j) % (*fact) == 0) {
                    chmin(dp[i + 1][0], dp[i][0] + (ll) b * abs(j));
                    chmin(dp[i + 1][2], dp[i][1] + (ll) b * abs(j));
                    chmin(dp[i + 1][2], dp[i][2] + (ll) b * abs(j));
                }
            }

            // remove
            chmin(dp[i + 1][1], dp[i][0] + a);
            chmin(dp[i + 1][1], dp[i][1] + a);
        }

        rep(i, 3)
            chmin(ans, dp[n][i]);
    }

    cout << ans << endl;

    return 0;
}
```


## まとめ
参加者は3420人。

- A: 3178/3268
- B: 2415/3190
- C: 512/1595
- D: 7/72
- E: 1/100

なかなか罠の多いセットだった。
