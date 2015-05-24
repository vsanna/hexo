title: "Codeforces #278 (Div. 1) A"
date: 2015-05-24 17:20:50
tags:
- esplo77
- Codeforces
- C++11
---

# A. Fight the Monster

何かに使えそう

## 問題

<http://codeforces.com/problemset/problem/487/A>

Yangさんがモンスターと戦う。
それぞれ3つのパラメーター、HP、ATK、DEFを持ち、モンスターのHPが0になった時にYangのHPが0でなければYangの勝ちになる。

YangのHPへのダメージは、max(0, (敵のATK - YangのDEF)) で求められる。
敵へのダメージも同様。
それぞれのダメージ反映は同時に行われ、これをどちらかのHPが0になるまで繰り返す。

Yangの各パラメーターはお金で上げられる。
初期パラメーター、およびそれぞれのパラメーターを1上げるのに必要なお金が与えられるので、Yangが勝つために必要な最小のお金を求めよ。

（変数名などは問題文を参照のこと）

$$
1 \leq HP_y, ATK_y, DEF_y, HP_m, ATK_m, DEF_m, h, a, d \leq 100
$$

## 考え方

まず、それぞれの値が取り得る範囲を考える。
それぞれの変数は1～100なので、$HP_m \leq 100$、$DEF_m \leq 100$。
これにより、$ATK_y$は200を超える必要はない（最初のターンで敵のHPを0にできる）。
また、$ATK_m \leq 100$なので、$DEF_y$は100を超える必要はない（ダメージを食らわなくなる）。
$HP_y$の上限は、食らうダメージの最大が100、自分の与えるダメージの最小が1（100ターンかかる）ため、$100 \times 100 = 10^4$となる。

まとめると、以下のような制約がある（パラメーターの追加分をaとする）。
$$
1 \leq HP_y + HP_a \leq 10^4 \\\
1 \leq ATK_y + ATK_a \leq 200 \\\
1 \leq DFE_y + DFE_a \leq 100
$$

この条件内を全探索して、勝てるかどうか判定し、勝てる場合は必要経費を求めれば良い。

### 制限時間

見て分かる通り、このまま全探索するとTLEする。
ATKとDEFが定まった状態であれば、$HP_y + HP_a$はある一点を境に勝敗が分かれる。
HPがいくら高かろうが、敵を倒した後の余ったHPが増えるだけなためである。

というわけで、二分探索が使える。
ループの順番を変え、一番内側でHPを求めるよう変形した後、にぶたんに変形すれば$10^4 \cdot log(10^4) = 10^5$回程度のループで済む。

## コード

```C++
int damage(int atk, int def) { return max(0, atk - def); }
bool isWin(int hpy, int atky, int defy, int hpm, int atkm, int defm) {
    int myd = damage(atkm, defy);
    int md = damage(atky, defm);

    if( myd != 0 ) {
        int turn = hpy / myd;
        if(turn == 0) return false;
        if(hpy % myd == 0)
            turn--;
        return (turn*md >= hpm);
    }
    else if(md != 0)
        return true;
    else
        return false;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int HPy, ATKy, DEFy;
    int HPm, ATKm, DEFm;
    int h, a, d;
    cin >> HPy >> ATKy >> DEFy >> HPm >> ATKm >> DEFm >> h >> a >> d;

    int res = INT_MAX;

    rep(j, 201) {
        rep(k, 101) {

            int l = -1;   // possible max
            int r = 20000;   // impossible min
            while (r - l > 1) {
                int i = (r + l) / 2;

                if (isWin(HPy + i, ATKy + j, DEFy + k, HPm, ATKm, DEFm)) {
                    res = min(res, i * h + j * a + k * d);
                    r = i;
                }
                else
                    l = i;
            }
        }
    }

    cout << res << endl;

    return 0;
}
```

## 結果
15ms, 4KB

解法考える -> 高速化する、という王道的問題で楽しい。