title: "Codeforces #335 Div.2 A-D"
date: 2015-12-12 02:06:22
tags:
- esplo77
- Codeforces
- C++11
---

## A. Magic Spheres

### 解き方
それぞれの色で余る数を求め、変換可能な個数（余り/2）を保持する。
また、それぞれの色での不足数を足し、変換可能な個数と大小を比較すればよい。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int a[3], b[3];
    rep(i,3)
        cin >> a[i];
    rep(i,3)
        cin >> b[i];

    int rest = 0;
    int sho = 0;

    rep(i,3) {
        rest += max( (a[i] - b[i]) / 2, 0);
        sho += max(b[i] - a[i], 0);
    }
    if(rest >= sho)
        cout << "Yes" << endl;
    else
        cout << "No" << endl;

    return 0;
}
```

## B. Testing Robots
サンプル1の最後の数が何故6になるか分からず結構な時間を無駄にしてしまった。
与えられた命令をシミュレートし、以下の要領で答えを出力。

- まだ訪れたことのないマスに行った場合、1
- 訪れたことのあるマスに行った場合、0
- 最後の命令なら、まだ訪れたことのないマスの数を加算

コンテスト中は↓のコードで通したが、1を出力するたびに$x*y$から1を引いていくと、最後の値が簡単に求まる。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int x, y, x0, y0;
    cin >> x >> y >> x0 >> y0;
    --x0, --y0;
    vector<vi> field(x, vi(y, 0));

    string str;
    cin >> str;

    pii now = mp(x0, y0);
    field[x0][y0] = true;

    vi res;
    res.push_back(1);

    tr(it, str) {
        char c = *it;
        int dx = 0, dy = 0;
        if(c == 'U') dx = -1;
        else if(c == 'D') dx = 1;
        else if(c == 'L') dy = -1;
        else dy = 1;

        int nextx = now.first + dx;
        int nexty = now.second + dy;

        if( 0 <= nextx && nextx < x &&
                0 <= nexty && nexty < y) {
            if( field[nextx][nexty] )
                res.push_back(0);
            else {
                res.push_back(1);
                field[nextx][nexty] = true;
            }
            now = mp(nextx, nexty);
        }
        else
            res.push_back(0);
    }

    int no = 0;
    rep(i,x) {
        rep(j, y) {
            if(!field[i][j])
                no++;
        }
    }

    res[str.size()] += no;
    tr(it, res)
        cout << *it << " ";
    cout << endl;


    return 0;
}
```

## C. Sorting Railway Cars
パターンが掴めずかなり時間を使ってしまった。
原理を考えてみると、動かす必要の無い数は、「1ずつ増加している最長部分数列」。
サンプル1では、1 2 3は動かす必要がない事がわかる。

$1 \leq n \leq 10^5$なので、単純にLCSを求めると間に合わない。
「1ずつ増加する」という特性を活かすと、左から順に以下の漸化式を解けば$O(n)$で件のLCSが求まることが分かる。
$DP[p[i]] = 1 + DP[p[i]-1]$

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n;
    cin >> n;
    vi p(n);
    rep(i,n)
        cin >> p[i];

    map<int, int> dp;
    rep(i,n)
        dp[p[i]] = 1 + dp[p[i]-1];

    int mx = 0;
    REP(i,1,n+1)
        mx = max(mx, dp[i]);

    cout << n - mx << endl;

    return 0;
}
```

## D. Lazy Student
何故か通らない！と思ったら、想定外のケースがあった。

基本的には、コスト順にソートし順に処理する。
SMTなので、コストが小さい順に辿ると貪欲的に処理できる。
SMTに含まれるエッジはノード1から伸ばし、含まれないものは邪魔にならない位置に配置すれば良い。

- ソートする際は、SMTに含まれるエッジが先に来るようにして処理を簡易化
- 「邪魔にならない位置」は、「既にSMTに入っているノード同士をつなぐエッジ」
- -「邪魔にならない位置」に配置出来ない（SMTに含まれてしまう）場合、-1

最初は単純にコストが小さい順に1になるはずでしょ、と思っていたが、以下の様な正しいケースで落ちるため、上記処理が必要。

```text
input
5 7
2 1
3 1
3 0
4 1
5 0
6 0
7 1
output
1 2
1 3
2 3
1 4
2 4
3 4
1 5
```

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n,m;
    cin >> n >> m;
    vector<pair<pii,int>> ab(m);
    rep(i,m) {
        int a, b;
        cin >> a >> b;
        if(b == 1) b = 0;  // ソート順を変えるため
        else b = 3;
        ab[i] = mp(mp(a,b), i);
    }
    sort(all(ab));

    vector<pii> res(m);
    vector<bool> used(n+1, false);
    used[1] = true;
    int smt_edge = 2;
    int non_smt_now = 2;
    int non_smt_next = 3;

    rep(i,m) {
        if(ab[i].first.second == 0) { // SMT
            used[smt_edge] = true;
            res[ab[i].second] = mp(1, smt_edge++);
        }
        else {
            if(!used[non_smt_now] || !used[non_smt_next]) {
                cout << -1 << endl;
                return 0;
            }
            res[ab[i].second] = mp(non_smt_now, non_smt_next);

            if(non_smt_next - 1 == non_smt_now) {
                non_smt_now = 2;
                non_smt_next++;
            }
            else
                non_smt_now++;
        }
    }

    rep(i,m)
        cout << res[i].first << " " << res[i].second << endl;

    return 0;
}
```

## まとめ
総参加者は3057人。
以下、コンテスト中に解いた人数。

- A: 2098
- B: 921
- C: 872
- D: 251
- E: 7

Bが少し難しめで、C,Dが簡単めだった模様。

Cに気付くのが遅かった & Dのテストケースが漏れていたので、振るわない結果になった。
まずは過去問を解いて、コンテスト感を取り戻したい。