title: "Codeforces #341 A-E"
date: 2016-02-13 14:53:22
tags:
- esplo77
- Codeforces
- C++11
---

## A. Wet Shark and Odd and Even
[問題文](http://codeforces.com/contest/621/problem/A)

### 考え方
まず全てを足し合わせる。
結果が偶数ならそれが答えになり、奇数であれば、構成要素の中で最小の奇数を引くと偶数の答えになる。

### コード

```C++
int main() {
    int n;
    cin >> n;

    const int INF = (int) (1e9) + 10;

    ll sm = 0;
    int odd_num = 0;
    int min_odd = INF;
    vi nums(n);
    rep(i, n) {
        cin >> nums[i];
        sm += nums[i];
        if (nums[i] % 2 == 1) {
            odd_num++;
            chmin(min_odd, nums[i]);
        }
    };

    if (odd_num % 2 == 1)
        cout << sm - min_odd << endl;
    else
        cout << sm << endl;

    return 0;
}
```


## B. Wet Shark and Bishops
[問題文](http://codeforces.com/contest/621/problem/B)

### 考え方
i,jを行番号、列番号とすると、i+jは左斜めのマスが同じ値を取り、i-jは右斜めのマスが同じ値を取る。
各マスをたどり、i+jとi-jのそれぞれが同じ値を取る組み合わせを数えればよい。
i-jは負の値を取る可能性もあるため、999-i+jとして正の値にした。

### コード

```C++
int main() {
    int n;
    cin >> n;

    map<int, int> lb_num, rb_num;
    rep(i, n) {
        int x, y;
        cin >> x >> y;
        --x, --y;

        lb_num[x + y]++;
        rb_num[(999 - x) + y]++;
    }

    ll sum = 0;
    rep(i, 1000) {
        rep(j, 1000) {
            if (lb_num[i + j] != 0) {
                lb_num[i + j]--;
                sum += lb_num[i + j];
            }
            int rb = (999 - i) + j;
            if (rb_num[rb] != 0) {
                rb_num[rb]--;
                sum += rb_num[rb];
            }
        }
    }
    cout << sum << endl;

    return 0;
}
```


## C. Wet Shark and Flowers
[問題文](http://codeforces.com/contest/621/problem/C)

### 考え方
掛け合わせた値が素数pで割り切れるのは、掛け合わせる要素にpを含む時のみ。
i,jが素数pの倍数になる確率をそれぞれ$p_i$、$p_j$とすると、「少なくともいずれかが素数になる確率」は、「1-どちらも素数でない確率」となる。
これは$1.0 - (1.0 - p_i)*(1.0 - p_j)$で求まる。
また、[l, r]にpを含む場合の数は$\frac{r - (l-1)}{p}$となる。

上記の計算を隣り合うペアで行うことで、2000円が手に入る確率が求まる。
期待値はそれぞれを足し合わせることで全体の期待値になる（線形性がある）ので、全ペアで期待値を計算し足し合わせれば答えになる。

### コード

```C++
int main() {
    int n, p;
    cin >> n >> p;

    vector<double> poss(n, 1.0);
    rep(i, n) {
        int l, r;
        cin >> l >> r;
        ll numerator = (r - l + 1);
        ll num = (r / p) - ((l - 1) / p);

        poss[i] = (1.0 - (double) (num) / (double) (numerator));
    }

    double expect = 0.0;
    rep(i, n) {
        int next_i = (i + 1) % n;
        expect += 2000.0 * (1.0 - poss[i] * poss[next_i]);
    }

    cout << expect << endl;

    return 0;
}
```


## D. Rat Kwesh and Cheese
[問題文](http://codeforces.com/contest/621/problem/D)

### 考え方
まず、$(a^b)^c$と$(a^c)^b$は同じなので、前者のみ採用する。
次に、大きな値（$200^{200^{200}}$）を比較するため、それぞれlogを取る。
$a < b$に対し$loga < logb$が成り立つので、logを取って比較する。

これだけだと簡単な問題だが、1回logを取ってもまだ$200^{200}$という大きい数字なので、更にもう一度logを取る必要がある
（C++ではlong doubleに収まってしまうらしい……。他の言語だともっと楽に扱えそう）。

2回目のログを取る際は、$a <= 1$のとき$loga <= 0$となり、0以下のlogは取れないため注意が必要。
ここで、それぞれの式のlogを2回取ってみると、以下の式となる。

$$
loglog(x^{y^z}) = z \ast log(y) + loglog(x) \\\
loglog(x^{z^y}) = y \ast log(z) + loglog(x) \\\
loglog((x^y)^z) = log(y \ast z) + loglog(x)$$


このように、loglogが取られるのはxのみなので、$x > 1.0$のときはこの計算が行える。
そして、x > 1.0 && y <= 1.0 && z <= 1.0だと、上記3つのいずれかが答えになる（logを1度だけ取った式で比較をすると、他は負の値になるため）。

上記計算をx,y,zそれぞれを基準にして行い、値を求める。
コーナーケースであるx<=1.0 && y <= 1.0 && z <= 1.0では、小さな値に収まるため、logを1回だけ取って比較をすればよい。

下記コードでは対称性を利用して関数に押し込んでいる。
少しわかりにくい条件分岐になってしまった。

### コード

```C++
vector<double> res(12, -INF);

void assign(int start, double x, double y, double z) {
    res[start] = z * log(y) + log(log(x));
    res[start + 1] = y * log(z) + log(log(x));
    res[start + 2] = log(y * z) + log(log(x));
}
void normal_assign(int start, double x, double y, double z) {
    res[start] = pow(y, z) * log(x);
    res[start + 1] = pow(z, y) * log(x);
    res[start + 2] = y * z * log(x);
}
string ans_str[] = {
        "x^y^z",
        "x^z^y",
        "(x^y)^z",
        "",
        "y^x^z",
        "y^z^x",
        "(y^x)^z",
        "",
        "z^x^y",
        "z^y^x",
        "(z^x)^y",
        ""
};

int main() {
    double x, y, z;
    cin >> x >> y >> z;

    if (x > 1.0)
        assign(0, x, y, z);
    if (y > 1.0)
        assign(4, y, x, z);
    if (z > 1.0)
        assign(8, z, x, y);

    if (x <= 1.0 && y <= 1.0 && z <= 1.0) {
        normal_assign(0, x, y, z);
        normal_assign(4, y, x, z);
        normal_assign(8, z, x, y);
    }

    int min_i = 0;
    double ans = -INF;
    rep(i, 12) {
        if (chmax(ans, res[i]))
            min_i = i;
    }

    cout << ans_str[min_i] << endl;

    return 0;
}
```


## E. Wet Shark and Blocks
[問題文](http://codeforces.com/contest/621/problem/E)

### 考え方
dp[d][i] := d個数値を取り出した時、iが作れるパターン数
とおく（iはmod x済み）。

値aを使い、aの個数を|a|とするとき、このDPは以下のように計算される。
$dp \bigl[ d+1 \bigr] \bigl[ (10 * i + a) \pmod{x} \bigr] = dp \bigl[ d \bigr] \bigl[ i \bigr] \ast |a|$

dp表は十分に小さいが、桁数bが最大10^9と大きいので、行列で漸化式を表し、繰り返し二乗法を使って高速化する。
i行目j列目の要素は、$(10*j + t) \bmod{x} = i$となる要素の個数を設定する。
mod 4を例に出すと、mat[0][0] = |0|、mat[0][1] = |2| （12 % 4 = 0)、mat[0][2] = |0| (20 % 4 = 0)、mat[0][3] = |2| (32 % 4 = 0)となる。
これで行列の1行目の要素が埋まるので、下の行に向けて1ずつ足した要素の個数を取れば楽に求まる。

### コード
```C++
int x;

vvi mul(vvi &A, vvi &B) {
    vvi C(A.size(), vi(B[0].size()));
    rep(i, A.size()) {
        rep(k, B.size()) {
            rep(j, B[0].size()) {
                C[i][j] = ((ll) C[i][j] + ((ll) A[i][k] * B[k][j]) % MOD) % MOD;
            }
        }
    }
    return C;
}

vvi pow(vvi A, ll n) {
    vvi B(A.size(), vi(A.size()));
    rep(i, A.size())
        B[i][i] = 1;
    while (n > 0) {
        if (n & 1) B = mul(B, A);
        A = mul(A, A);
        n >>= 1;
    }
    return B;
}

int main() {
    int n, b, k;
    cin >> n >> b >> k >> x;

    vi nums(x, 0);
    rep(i, n) {
        int a;
        cin >> a;
        nums[a % x]++;
    }

    vvi mat(x, vi(x, 0));
    rep(c, x) {
        int d = (c * 10) % x;
        int m = (x - d) % x;
        rep(r, x) {
            mat[r][c] = nums[m % x];
            m++;
        }
    }

    vvi init(nums.size(), vi(1));
    rep(i, nums.size())
        init[i][0] = nums[i];

    vvi An = pow(mat, b - 1);
    vvi res = mul(An, init);
    cout << res[k][0] << endl;

    return 0;
}
```


## まとめ
参加者は3877人。

- A: 3308/3810
- B: 2458/3209
- C: 922/1370
- D: 40/540
- E: 199/310

E問題が他の回に比べてかなり簡単だった。
D問題は普段あまり見ないタイプの問題だったが、言語による難易度の差が大きそう。
