title: "Codeforces Round #311 C"
date: 2015-07-02 22:27:23
tags:
- esplo77
- Codeforces
- C++11
---

# C. Arthur and Table

あと少しで解けず。

## 問題

椅子が安定しているとは、最大の長さを持つ脚が、脚の数の過半数を占める状態を言う。
$i$番目の脚の長さを$l_i$、その脚を取り除くコストを$d_i$とするとき、椅子を安定させる最小のコストを答えよ。

$$
1 \leq n \leq 10^5 \\\
1 \leq l_i \leq 10^5 \\\
1 \leq d_i \leq 200
$$

## 考え方
最大の脚の数が$n$のとき、それ以外の脚の数が$n-1$以下であれば条件を満たす。
最大の脚以外の長さはどうでもいいので、「最大の脚の長さを決め打ち」「残りの脚をコストが低い順に除去」すればよい。

### ナイーブな方法
1. 脚の長さを1～$10^5$まで順にループし、最大の脚の長さ(length_max)とする
2. lenght_maxより短い脚の本数をカウント
3. (2.で求めた値) - (最大の脚の本数 - 1) = 取り除く本数
4. 事前にコスト順に脚をソートしておき、「最大の脚の長さ未満」の長さを持つ脚を見つけ、取り除く本数に至るまで繰り返す
5. 4.にlength_maxより大きい脚を除外するコストを加える
6. ループした中で、5.の最小値を求める

これだと、1.に$O(l)$、2.に$O(n)$、4.に$O(n)$、5.に$O(n)$かかる。
トータルで$O(n \cdot (l + n) )$。
当然間に合わないので、高速化が必要。

### 工夫1
長い方からループを始めると、処理がスッキリする。
4,5番で何かが起きているが、後述する。

1. 最大の脚の長さを求める
2. length_max -> 1までループ（iとする）
3. 除外する脚の候補から、iの長さを持つ脚を消す（除外する脚の候補には、length_max未満の長さの脚しかない）
4. 除外する脚の候補から、$n - (ss_lc[i]\cdot2 - 1)$本コストが小さい順に除外
5. 4.にss_d[i]を加える
6. ループした中で5.の最小値を求める

これで求まる。
3はlength_maxを持つ脚の数だけループするので、全ループ合わせて$n$だけ回る。
脚の候補はコスト順に並んでいるので、長さで目的の脚を消すのは大変（$O(n)$かかる）。

4はループ不要で計算できている。
ss_lcは、各脚の長さのsuffix sum。
$i$以上の長さを持つ脚の数を記録している。
例えば、1の長さが2本、2の長さが3本、3の長さが1本とすると、
ss_lc[1]=6
ss_lc[2]=4
ss_lc[3]=1
となる。

また、5で使われているss_d[i]もコストのsuffix sum。
4.と同様の考え方で、ループ不要で計算できる。

この方法だと、$O(n^2)$になる。結構良くなった。

### 工夫2
3.の手順を早くしないと結局間に合わない。
ここで、dが小さいことを利用。
hist[d]をコストがdの脚の個数を記録した配列とする。

以下のように、3と4を置き換える。

3. 長さがlength_maxの脚を取り出し、hist[コスト]をデクリメントするだけ
4. hist[1]～hist[200]までループをし、必要な個数除外しコストを計算

これで求まる。
$O(n \cdot d)$となり、十分間に合う。

要は、ヒストグラムを使って高速に計算しよう、ということ。

## コード

```C++
const int MAX = (int) (1e5) + 2;
int length_count[MAX];
int length_d[MAX];

int ps_dd[MAX];
int ps_lc[MAX];

vi length_map[MAX];


int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n;
    cin >> n;

    vi l(n);
    rep(i, n) {
        cin >> l[i];
        length_map[l[i]].push_back(i);
    }

    vi hist_d(201, 0);
    vi d(n);
    rep(i, n) {
        cin >> d[i];
        hist_d[d[i]]++;
    }

    vector<pii> dl(n);
    rep(i, n)
        dl[i] = mp(d[i], l[i]);


    memset(length_count, 0, sizeof(length_count));
    memset(length_d, 0, sizeof(length_d));

    memset(ps_dd, 0, sizeof(ps_dd));
    memset(ps_lc, 0, sizeof(ps_lc));

    priority_queue<pii, vector<pii>, greater<pii> > length_que;

    int length_max = 0;
    rep(i, n) {
        int t = l[i];
        length_que.push(mp(d[i], l[i]));
        length_count[t]++;
        length_max = max(t, length_max);
        length_d[t] += d[i];
    }

    // prefix sums
    {
        int tmp = 0;
        for (int i = MAX - 1; i >= 1; i--) {
            tmp += length_count[i];
            ps_lc[i - 1] = tmp;
        }
    }
    {
        int tmp = 0;
        for (int i = MAX - 1; i >= 1; i--) {
            tmp += length_d[i];
            ps_dd[i - 1] = tmp;
        }
    }

    if (length_count[length_max] > n / 2) {
        cout << 0 << endl;
        return 0;
    }

    sort(all(dl));

    int res = INT_MAX;

    for (; length_max > 0; --length_max) {
        if (length_map[length_max].empty())
            continue;

//        cerr << "*hist_d " << length_max << " " << length_map[length_max] << endl;
        for (int t: length_map[length_max])
            hist_d[d[t]]--;

        int rm_count = n - (length_count[length_max] * 2 - 1) - ps_lc[length_max];
//        cerr << rm_count << endl;
        int cost = 0;
        rep(i, 201) {
            if (rm_count <= 0)
                break;

            if (hist_d[i] < rm_count) {
//                cerr << "hist : " << i << " " << hist_d[i] << endl;
                cost += hist_d[i] * i;
                rm_count -= hist_d[i];
            }
            else {
                cost += rm_count * i;
                break;
            }
        }

        res = min(res, ps_dd[length_max] + cost);
//        cerr << length_max << " : " << res << "  orig: " << ps_dd[length_max] + cost<< endl;
    }

    cout << res << endl;

    return 0;
}
```