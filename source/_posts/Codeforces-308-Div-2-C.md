title: "Codeforces #308 Div.2 C"
date: 2015-06-20 00:19:27
tags:
- esplo77
- Codeforces
- C++11
---

# C. Vanya and Scales

とりあえず書いて出したら、案の定Hackされた。

## 問題

重さが$w^0, w^1, ..., w^100$の100個の分銅がある。
重さ$m$のおもりを秤の左皿に置く。
分銅を左右の更に置いて釣り合うことができれば"YES"、できなければ"NO"を出力せよ。

## 考え方

左の皿に重さAとBの分銅、右の更に重さCとDの分銅を置いたとすると、$m+A+B=C+D$が成立する。
これを式変形すると、$m=C+D-A-B$となる。
つまり、mが分銅の組み合わせ（+, -, 使わないのいずれか）で作れれば良い。

### BAD END

ダメな解法を一応書いておく。

最大の重さを持つ分銅を最初に定める。
その分銅は+として使い、それ未満の重さを持つ分銅は+, -, 使わないのどれでもよいとする。
これを全探索し、mに一致するかどうかを求める。
再帰で求めることができるが、当然$O(2^n)$でTLEする。

### GOOD END

まず、mをwの累乗で表記する。
たとえば、$w=3$、$m=25$であれば、$m=2\cdot3^2+2\cdot3^1+1\cdot3^0$となる。
分銅はそれぞれ1回しか使えないので、$x\cdot3^k$の$x$が0,1であれば何もしなくて良いが、2以上であれば変形して0,1にできるかを調べる必要がある。
$x\cdot3^k = 3^{k+1} - (3-x)\cdot3^k$という式変形を施し、全てのxが0,1になればmが作れるということになる。
kが小さい方がらループを回し、$O(n)$で求まる。

$w^{100}$とかは求めるだけで死んでしまうので、mを初めて超えた時の指数を記録しておき、そこまでしか回さないような工夫が必要。

## コード

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int w,m;
    cin >> w >> m;

    const int MAX = 102;
    vi digits(MAX,0);

    int maxK = 0;
    ll tmpW = 1;
    rep(i,MAX) {
        if(m < tmpW) {
            maxK = i;
            break;
        }
        tmpW *= w;
    }

    int tmpM = m;
    for(int i = maxK-1; i>=0; --i) {
        tmpW /= w;
        digits[i] = (tmpM / tmpW);
        tmpM -= digits[i] * tmpW;
    }

    rep(i, MAX-1){
        if(digits[i] == 0 || digits[i] == 1)
            continue;
        digits[i+1]++;
        digits[i] = w - digits[i];

        if(digits[i] != 0 && digits[i] != 1){
            cout << "NO" << endl;
            return 0;
        }
    }

    cout << "YES" << endl;
    return 0;
}
```

## 結果
31 ms, 0 KB

簡単なのに何故解けなかったのか。
本番は何が起きるかわからない。
