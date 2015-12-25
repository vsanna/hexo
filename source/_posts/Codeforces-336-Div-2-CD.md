title: "Codeforces #336 Div.2 A-D"
date: 2015-12-25 17:03:22
tags:
- esplo77
- Codeforces
- C++11
---

## A. Saitama Destroys Hotel
### 考え方
各フロアでの最大待ち時間を計算（おそらくInputのフロアはユニークなので、単純に配列に記録してゆけばよい）。
上のフロアから順に降りて行き、待ち時間と現在の時間を比較し必要なら待てばよい。

コンテスト中は↓のコードで通したが、0階の入力は与えられないので最後のループ1回と減産はムダ。

```C++
int main() {
    int n,s;
    cin >> n >> s;

    vi pas(s+1, 0);
    rep(i,n) {
        int f,t;
        cin >> f >> t;
        pas[f] = max(pas[f], t);
    }
    int now = 0;
    for(int i=s; i>=0; i--) {
        if(now <= pas[i])
            now = pas[i];
        now++;
    }
    now--;

    cout << now << endl;

    return 0;
}
```

## B. Hamming Distance Sum
### 考え方
単純に部分文字列を列挙して計算すると、予想通りTLEする（計算量は|b|の2乗くらい？）。
ここで、文字列A: ab、文字列B: cdefgとし、比較の様子を図示すると以下のようになる。

比較その1
- ab
- cd
比較その2
- ab
- de
比較その3
- ab
- ef
比較その4
- ab
- fg

上記4回の比較が行われるが、このときaは(c,d,e,f)、bは(d,e,f,g)という連続した文字との比較しか行われないことがわかる。
文字は0/1なので、0/1の個数を累積和で保持しておき、自分と異なる要素が何回出現するかを計算すればよい。

```C++
int main() {
    string a,b;
    cin >> a >> b;

    vi accum(b.size()+1, 0);
    rep(i,b.size()){
        accum[i+1] = accum[i];
        if(b[i] == '1')
            accum[i+1]++;
    }

    ll res = 0;
    rep(i, a.size()) {
        int ti = b.size()+i-a.size()+1;
        int onenum = accum[ti] - accum[i];
        int zeronum = (ti-accum[ti]) - (i-accum[i]);

        if(a[i] == '1')
            res += zeronum;
        else
            res += onenum;
    }

    cout << res << endl;

    return 0;
}
```


## C. Chain Reaction
### 考え方
問題が少しややこしい。
押したビーコンは破壊されず、その左側で$b_i$の範囲内にあるビーコンだけ破壊されることを把握しておく。

破壊される可能性があるかを判別するのは厄介なので、「破壊されない」という情報を記録してDPする。
dp[i] := iの位置にあるビーコンを作動させた時、0..i の範囲で最大の不発ビーコン数

このように定義すると、爆発させる処理をきれいに扱うことができる。
ビーコンの数からdp配列中の最大要素を引くと、最小の爆発数が出てきて答えになる。
漸化式などはコードで。

```C++
int main() {
    int n;
    cin >> n;

    int mx = 0;
    map<int, int> ab;
    rep(i,n) {
        int a, b;
        cin >> a >> b;
        mx = max(mx, a);
        ab[a] = b;
    }

    // dp[i] := 0..i までで不発の数
    vi dp(mx+1, 0);
    rep(i,mx+1){
        if(ab[i] > 0) {  // 爆弾あり
            if (i - ab[i] - 1 < 0)
                dp[i] = 1;
            else
                dp[i] = dp[i-ab[i]-1] + 1;
        }
        else {  // なし
            if(i > 0)
                dp[i] = dp[i-1];
        }
    }

    cout << n - *max_element(all(dp)) << endl;

    return 0;
}
```


## D. Zuma
愚直に回文を見つけようとすると大変なので、DPで開始位置と終了位置それぞれで「回文を考慮した最小処理回数」を保持しておく。
例えばサンプル1では、以下のようになる。

dp[0][0] = dp[1][1] = dp[2][2] = 1 （1文字は1回の処理で取り除ける）
dp[0][1] = dp[0][0] + dp[1][1] = 2
dp[1][2] = dp[1][1] + dp[2][2] = 2
dp[0][2] = dp[1][1] = 1 （端っこが同じ文字なので、より小さい範囲のdp配列の値を流用できる）

このように部分問題に帰着させるような処理を行いDP配列を埋めて行けば、最終的にdp[0][n-1]が答えになる。

```C++

int main() {
    int n;
    cin >> n;
    vi c(n);
    rep(i,n) cin >> c[i];

    // dp[start][end] = count
    vector<vi> dp(n, vi(n, INF));

    REP(len, 1, n+1){
        rep(start, n-len+1){
            int end = start + len - 1;
            if(len == 1)
                dp[start][end] = 1;
            else {
                if(c[start] == c[end]) {
                    if(len == 2)
                        dp[start][end] = 1;
                    else
                        dp[start][end] = min(dp[start][end], dp[start+1][end-1]);
                }
                REP(i, start, end) {
                    dp[start][end] = min(dp[start][end], dp[start][i] + dp[i + 1][end]);
                }
            }
        }
    }

    cout << dp[0][n-1] << endl;

    return 0;
}
```


## まとめ
参加者は3205人。

- A: 2685/3085
- B: 1367/2450
- C: 858/1226
- D: 195/366
- E: 8/51

今回はB問題が少し難しめ、D問題が簡単めだった。
自分はC問題が全く思い浮かばず、Dを解きにいった。
解法は合っていたが、ループがおかしいようで何故かWA。
コンテスト後にテストケースを見てもよくわからず、長さでループしたらすんなり通った。
原因不明で気持ち悪いが、なんにせよかなり勿体ないことをした……。
レーティングが下がりっぱなしだが、問題の手ごたえは掴んできているので、そろそろ100位以内に入りたい。

なお、今回はワンパンマン回だった。
てーきゅう回はまだかな？
……いつかは作問者になりたい！
