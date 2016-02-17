title: "Codeforces 8VC Venture Cup 2016 - Elimination Round A-D"
date: 2016-02-17 23:50:00
tags:
- esplo77
- Codeforces
- C++11
---

## A. Robot Sequence
[問題文](http://codeforces.com/contest/626/problem/A)

### 考え方
連続する部分文字列を使って元の位置に戻るかを判定する問題。
$n \leq 200$と小さいので、愚直に$O(n^3)$の方法で解ける。

### コード

```C++
int main() {
    int n;
    cin >> n;
    string str;
    cin >> str;

    int ans = 0;
    rep(i, n) {
        REP(j, i+1, n) {
            int h = 0;
            int v = 0;
            REP(k, i, j+1) {
                if(str[k] == 'U') v++;
                else if(str[k] == 'D') v--;
                else if(str[k] == 'L') h--;
                else h++;
            }
            if(h == 0 && v == 0)
                ans++;
        }
    }
    cout << ans << endl;
    
    return 0;
}
```

## B. Cards
[問題文](http://codeforces.com/contest/626/problem/B)

### 考え方
1. 違う色のカードを1枚ずつ使って、それ以外の色のカードに変換する
2. 同じ色のカード2枚を1枚にする

という2つの変換がある。
2番目の変換を使うと、その色の枚数は自由に1枚にまで減らすことができる。
また、1番目の変換と合わせ、少なくとも2色2枚以上カードがあれば、全ての色が作れる。

1色だけの場合はそれしか作れないこと、1枚と2枚と0枚であれば、2枚の色以外が作れることに注意して条件列挙をすれば良い。

それぞれの色が作れるかを順にチェックしてゆくとわかりやすい。
出力はアルファベット順にしないといけないので注意。

### コード
```C++
int main() {
    int n;
    cin >> n;
    string str;
    cin >> str;

    vi col(3, 0);
    rep(i, n) {
        if(str[i] == 'B') col[0]++;
        else if(str[i] == 'G') col[1] ++;
        else col[2]++;
    }

    string color_str = "BGR";
    string ans = "";
    rep(i, 3) {
        int cur = col[i];
        int next = col[(i+1) % 3];
        int nnext = col[(i+2) % 3];

        if( (next == 0 && nnext == 0) ||
            (next >= 1 && nnext >= 1) ||
            (cur >=1 && (next >= 2 || nnext >= 2) ) )
            ans += color_str[i];
    }
    cout << ans << endl;
    
    return 0;
}
```


## C. Block Towers
[問題文](http://codeforces.com/contest/626/problem/C)

### 考え方
答えの数をXと置くと、満たすべき条件は以下の3つのみ。
これを満たすかどうかを全探索すれば良い。
3番目の式を変形して、答えはおおよそ$7 \times n$に収まるので、ざっくりと$10 \times n$まで回せば求まる。

$$
n \leq \lfloor \frac{X}{2} \rfloor \\\
m \leq \lfloor \frac{X}{3} \rfloor \\\
n+m \leq \lfloor \frac{X}{2} \rfloor + \lfloor \frac{X}{3} \rfloor - \lfloor \frac{X}{6} \rfloor
$$

### コード
```C++
int main() {
    int n,m;
    cin >> n >> m;

    rep(i, (int)(1e7)+10) {
        if( n <= i/2 &&
                m <= i/3 &&
                n+m <= i/2 + i/3 - i/6) {
            cout << i << endl;
            return 0;
        }
    }

    return 0;
}
```

## D. Jerry's Protest
[問題文](http://codeforces.com/contest/626/problem/D)

### 考え方
方針として、場合の数を求め、全体の数で割って確率を算出する。

まずは取りうる差分を求める。
どちらかのプレイヤーから見ると、勝つ場合は正で負ける場合は負になるが、そこを区別する必要が無いため、取りうる値とパターン数だけ考えれば良い。
n個の数字から2つを取るので、nC2だけ存在する。

1回の差分パターンが計算された後、
1回目の差分 + 2回目の差分 < 3回目の差分
となる差分が必要なため、1回目の取りうる差分と2回目の取りうる差分を全探索して場合の数を求める。
なお、3回目の差分の数（ある値より大きいパターン数）は、累積和を使って高速に求めることができる。

全体の数はnC2の3乗になるので、場合の数をこれで割って答えを出す。

### コード
```C++
const int MAX_A = 5000 + 5;
int main() {
    int n;
    cin >> n;
    vi a(n, 0);
    rep(i, n) cin >> a[i];

    vi diffs(MAX_A, 0);
    rep(i, n) {
        REP(j, i+1, n) {
            int d = abs(a[i] - a[j]);
            diffs[d]++;
        }
    }

    // suffixの累積和
    vll accum(MAX_A, 0);
    for(int i=MAX_A-2; i>=0; --i) {
        accum[i] = accum[i+1] + diffs[i];
    }

    ll pattern = 0;
    REP(i, 1, MAX_A) {
        REP(j, 1, MAX_A) {
            if(i+j+1 < MAX_A)
                pattern += (ll)diffs[i] * diffs[j] * accum[i+j+1];
        }
    }

    int total = n * (n - 1) / 2;
    cerr << total << endl;
    cout << (double)pattern / total / total / total << endl;
    
    return 0;
}
```

## まとめ
参加者は2879人。

- A: 2589/2714
- B: 2073/2439
- C: 1347/2334
- D: 880/1124
- E: 134/449
- F: 129/191
- G: 11/23
