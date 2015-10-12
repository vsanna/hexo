title: "Ari-practice solve 9"
date: 2015-09-28 23:43:31
tags:
- esplo77
- C++
- 蟻本
---


# 蟻本練習問題を解いた (9)

## AOJ0005 - GCD and LCM
基本問題。
ただ書くだけ。

```C++
int gcd(int a, int b) {
    if(a == 0)
        return 1;
    if(b == 0)
        return a;
    return gcd(b, a%b);
}
int lcm(int a, int b) {
    return a / gcd(a,b) * b;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int a,b;
    while(cin>>a>>b) {
        cout << gcd(a,b) << " " << lcm(a,b) << endl;
    }

    return 0;
}
```

## POJ2429 - GCD & LCM Inverse
[ここを見た](http://quiz.fuqinho.net/blog/2012/06/12/poj-2429-gcd-and-lcm-inverse/)
素因数分解すれば解の候補がでるが、値が大きい。
乱択アルゴリズムであるミラーラビンを使ったりして頑張って解く模様。
練習問題のレベルを超えているので、保留。


## POJ1930 - Dead Fraction
与えられる数の循環する桁数を仮定して全て試し、その中で分母が最小になるものを求めれば良い。

0.abcd...という循環小数を少数に直すのは、abcd/9999のように、10の桁数乗-1で割れば良い。
もちろん、循環しない部分は普通に10の乗数で割れば良い。

```C++
const ull INF = 1ll << 63;
ull gcd(ull a, ull b) {
    if(b == 0)
        return a;
    return gcd(b, a%b);
}
ull s2i(string str) {
    ull num = 0;
    istringstream ss(str);
    ss >> num;
    return num;
}
void div(ull& a, ull& b) {
    ull d = gcd(a,b);
    a /= d, b/= d;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    string line;
    while(cin >> line, (line!="0")) {
        ull min_numer = INF;
        ull min_denom = INF;
        string str = line.substr(2, line.size()-5);

        rep(i, str.size()) {
            ull non_loop_numer, non_loop_denom, loop_numer, loop_denom;
            ull pow_loop = fpow(10, str.size() - i);
            ull pow_non_loop = fpow(10, i);

            {  // create two fractions, loop and non-loop
                if (i == 0)
                    non_loop_numer = 0;
                else
                    non_loop_numer = s2i(str.substr(0, i));
                non_loop_denom = pow_non_loop;
                loop_numer = s2i(str.substr(i));
                loop_denom = (pow_loop - 1) * pow_non_loop;

                div(non_loop_numer, non_loop_denom);
                div(loop_numer, loop_denom);
            }

            {   // add two fractions
                ull numerator, denominator;
                numerator = non_loop_numer * loop_denom + non_loop_denom * loop_numer;
                denominator = non_loop_denom * loop_denom;

                div(numerator, denominator);

                if (min_denom > denominator)
                    min_numer = numerator, min_denom = denominator;
            }
        }

        cout << min_numer << "/" << min_denom << endl;
    }

    return 0;
}
```


## AOJ0009 - Prime Number
篩を作ってカウントするだけ。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    const int MAX_N = 1000000;
    bool sieve[MAX_N];
    fill(sieve, sieve+MAX_N, true);
    sieve[1] = false;

    for(int i=2; i*i <= MAX_N; ++i) {
        for(int j=i*i; j < MAX_N; j += i) {
            sieve[j] = false;
        }
    }

    vi prime_num(MAX_N, 0);
    REP(i, 1, MAX_N){
        prime_num[i] = prime_num[i-1];
        if(sieve[i])
            prime_num[i]++;
    }

    int n;
    while(cin >> n) {
        cout << prime_num[n] << endl;
    }

    return 0;
}
```


## POJ3126 - Prime Path
一見難しそうに見えるが、意外と簡単。
素数を列挙して辺を作り、グラフをたどれば良い。
グラフといってもコストが全て同じなので、ただの幅優先探索で良い。
$O(10^8)$近いのでTLEしそうだったが、意外と大丈夫だった。

```C++
void split(int a, int* ary) {
    ary[0] = a / 1000;
    ary[1] = (a/ 100) % 10;
    ary[2] = (a/10) % 10;
    ary[3] = a % 10;
}
int diff(int a[4], int b[4]) {
    int count = 0;
    rep(i, 4)
        count += (int)(a[i] != b[i]);
    return count;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    const int MAX_N = 10010;
    vi is_prime(MAX_N, true);
    is_prime[0] = is_prime[1] = false;
    for(int i=2; i*i < MAX_N; ++i) {
        for(int j=i+i; j < MAX_N; j+=i) {
            is_prime[j] = false;
        }
    }

    map<int, vi> paths;
    REP(i, 1000, 10000) {
        REP(j, i, 10000) {
            if(is_prime[i] && is_prime[j]) {
                // 1文字だけ違うかどうか
                int spi[4];
                split(i, spi);
                int spj[4];
                split(j, spj);

                if(diff(spi, spj) == 1) {
                    paths[i].push_back(j);
                    paths[j].push_back(i);
                }
            }
        }
    }

    // bfs
    int _case;
    cin >> _case;

    while(_case--) {
        int from, to;
        cin >> from >> to;

        queue<pii> que;
        que.push(mp(from, 0));

        vector<bool> visited(100000, false);
        bool succeed = false;
        while(!succeed && !que.empty()) {
            pii now = que.front(); que.pop();
            if(visited[now.first])
                continue;

            if(now.first == to) {
                cout << now.second << endl;
                succeed = true;
                break;
            }

            visited[now.first] = true;

            tr(it, paths[now.first]) {
                que.push(mp(*it, now.second + 1));
            }
        }

        if(!succeed) {
            cout << "Impossible" << endl;
        }
    }

    return 0;
}
```


## POJ3421 - X-factor Chains
式がややこしく見えるが、結局素因数分解した数字を並び替える組み合わせ数を求めるだけ。
最大値が$2^{20}$なので、素因数の数は高々20。
20!は$10^{18}$くらいのようなので、long longなら入る。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    // 高々20程度
    vector<ll> fact(25, 0);
    fact[0] = fact[1] = 1;
    REP(i, 2, fact.size()) {
        fact[i] = fact[i-1] * i;
    }

    int n;
    while(cin >> n) {
        // 素因数分解
        map<int, int> factor;
        for(int i=2; i*i <= n; ++i) {
            while(n % i == 0) {
                factor[i]++;
                n /= i;
            }
        }
        if(n!=1)
            factor[n]++;

        vi elem_num;
        int sum = 0;
        tr(it, factor) {
            elem_num.push_back(it->second);
            sum += it->second;
        }

        ll numerator = fact[sum];
        ll denominator = 1;
        tr(it, elem_num)
            denominator *= fact[*it];
        cout << sum << " " << numerator / denominator << endl;
    }

    return 0;
}
```


## POJ3292 - Semi-prime H-numbers
$4x + 1$の数字だけがある元（？）を考え、その中で素数を定義している。
通常の篩を少しいじると、このような元が異なる場合でも対処できる。
最大値が$10^6$と決まっており、素数の数は$10^5$程度になるので、最大値以下になる素数の掛けあわせを全て列挙（$O(n\sqrt{n})$）しても間に合う。

H-semi-primeかどうかを保持するのは、累積和を取ることを考え、int配列で取っておくと楽。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    const int MAX_N = 1000020;
    vector<bool> is_h_prime(MAX_N, true);

    vi h_prime;
    for(int i=5; i < MAX_N; i += 4) {
        if(is_h_prime[i]) {
            h_prime.push_back(i);
            for (int j = 5; i * j < MAX_N; j += 4)
                is_h_prime[i * j] = false;
        }
    }

    vi num(MAX_N, 0);
    tr(it, h_prime) {
        for(vi::iterator it2=h_prime.begin(); it2 != h_prime.end() && (*it) * (*it2) < MAX_N; ++it2)
            num[(*it) * (*it2)] = 1;
    }

    REP(i, 1, num.size())
        num[i] += num[i-1];

    int n;
    while(cin >> n, n) {
        cout << n << " " << num[n] << endl;
    }

    return 0;
}
```


## POJ3641 - Pseudoprime numbers
いわゆるカーマイケル数かどうかを判定しろ、という問題。
繰り返し二乗法を使ったフェルマーテストと、試し割りで判定した結果を比べればおしまい。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int p, a;
    while(cin >> p >> a, (p|a)) {
        bool is_prime_fermat = fpow(a, p, p) == a;
        if(! is_prime_fermat) {
            cout << "no" << endl;
            continue;
        }

        bool is_prime = true;
        for(int i=2; i*i <= p; i++) {
            if(p % i == 0) {
                is_prime = false;
                break;
            }
        }
        if(p == 1)
            is_prime = false;

        if(is_prime)
            cout << "no" << endl;
        else
            cout << "yes" << endl;
    }

    return 0;
}
```

## POJ1995 - Raising Modulo Numbers
書くだけ。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int Z;
    cin >> Z;
    while(Z--) {
        int M, H;
        cin >> M >> H;

        ll res = 0;
        rep(i, H) {
            int a, b;
            cin >> a >> b;
            res += fpow(a, b, M);
            res %= M;
        }
        cout << res << endl;
    }

    return 0;
}
```


## 感想
数学的な問題は前提知識が必要なことが多く、かなり厄介な部類。
GCDは数学問題に限らず頻出なので、色々なパターンを身につけてゆきたい。

ついに次からは中級編。
アルゴリズム自体の難易度も実装難易度も上がると思われるので、じっくり腰を据えて解いてゆこうと思う。
