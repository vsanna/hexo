title: "C++競プロ用環境紹介"
date: 2015-08-09 02:59:09
tags:
- esplo77
- C++
- 環境
---

# C++環境
普段使っているC++の競プロ用環境を紹介。

## IDE
CLion (https://www.jetbrains.com/clion/)。
まともなIDEが殆ど無いC++において、彗星の如く現れた神環境。
有料（年$100）だが、その価値はあると思う。

## テスト
TopCoder以外はOnlineJudgeHelperを利用。
https://github.com/nodchip/OnlineJudgeHelper

フォルダ階層は以下のようにしている。

- abc001
    - abc001_1.cpp
    - abc001_2.cpp
    - oj.py
    - setting.json
    - r.sh
- oj.py
- setting.json
- create.sh
- r.sh

CLionでは、abc001内で新しいプロジェクトを作る。
最初にmain.cppが作られるが、CMakeListを書き換えて、A.cppなどをビルド対象にする。

create.shは、コンテスト用のフォルダを作るスクリプト。

```bash
#!/bin/sh
set -xue

mkdir -p $1
mv $2.cpp $1/
rsync -Aa oj.py $1/
rsync -Aa r.bat $1/
``````

例えば、コドフォ556（URLの末尾）のC問題を作る際は以下のコマンドを叩く。

```bash
$ ./create.sh 556 C
```

r.shはテストを実行＆サブミットするラッパー。
とりあえずコドフォ用。

```bash
$ python oj.py --codeforces $1 $2 $3
```

## テンプレート
とりあえずよく使うものを詰め込んだ。

```C++
#include <bits/stdc++.h>

using namespace std;

#define all(c) ((c).begin()), ((c).end())
#define dump(c) cerr << "> " << #c << " = " << (c) << endl;
#define iter(c) __typeof((c).begin())
#define tr(i, c) for (iter(c) i = (c).begin(); i != (c).end(); i++)
#define REP(i, a, b) for (int i = a; i < (int)(b); i++)
#define rep(i, n) REP(i, 0, n)
#define mp make_pair
#define fst first
#define snd second
#define pb push_back

typedef unsigned int uint;
typedef long long ll;
typedef unsigned long long ull;
typedef vector<int> vi;
typedef vector<vi> vvi;
typedef vector<double> vd;
typedef vector<vd> vvd;
typedef vector<string> vs;
typedef pair<int, int> pii;

const int INF = 1 << 29;
const double EPS = 1e-10;

double zero(double d) {
    return d < EPS ? 0.0 : d;
}

template<typename T1, typename T2>
ostream &operator<<(ostream &os, const pair<T1, T2> &p) {
    return os << '(' << p.first << ',' << p.second << ')';
}

template<typename T>
ostream &operator<<(ostream &os, const vector<T> &a) {
    os << '[';
    rep(i, a.size()) os << (i ? " " : "") << a[i];
    return os << ']';
}

// to avoid error on mingw
string toString(int i) {
    stringstream ss;
    ss << i;
    return ss.str();
}

const int M = 1000000007;
// a^k
ll fpow(ll a, ll k, int M) {
    ll res = 1ll;
    ll x = a;
    while (k != 0) {
        if ((k & 1) == 1)
            res = (res * x) % M;
        x = (x * x) % M;
        k >>= 1;
    }
    return res;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);



    return 0;
}
```

また、UnionFIndなどは必要に応じてスニペットを用意して使う。
