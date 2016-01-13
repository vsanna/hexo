title: "Ari-practice solve 14"
date: 2016-01-14 00:04:30
tags:
- esplo77
- C++
- 蟻本
---


# 蟻本練習問題を解いた (14)


##

##

## POJ2914 Minimum Cut
[ここを見た](http://blog.sina.com.cn/s/blog_6635898a0100qwil.html)
日本語どころか英語の解説も無かった。
仕方なく中国語のブログを眺めていたら、参考サイトを見つけた。
中国語で本文は読めなかったが、関数名だけ読めたため、答えに辿りつけた。

調べてみると、無向グラフの全域最小カットを$O(|V||E| + V^2log|V|)$で行える、Stoer-Wagner法なるアルゴリズムがあるらしい。
[英語版Wikipedia](https://en.wikipedia.org/wiki/Stoer%E2%80%93Wagner_algorithm)に記事があったため、利用した。
今回の問題では$O(n^3)$でも間に合うため、隣接行列で実装された方で通した。

```C++
pair<int, vi> GetMinCut(vvi &weights) {
    int N = weights.size();
    vi used(N), cut, best_cut;
    int best_weight = -1;

    for (int phase = N-1; phase >= 0; phase--) {
        vi w = weights[0];
        vi added = used;
        int prev, last = 0;
        for (int i = 0; i < phase; i++) {
            prev = last;
            last = -1;
            for (int j = 1; j < N; j++)
                if (!added[j] && (last == -1 || w[j] > w[last])) last = j;
            if (i == phase-1) {
                for (int j = 0; j < N; j++) weights[prev][j] += weights[last][j];
                for (int j = 0; j < N; j++) weights[j][prev] = weights[prev][j];
                used[last] = true;
                cut.push_back(last);
                if (best_weight == -1 || w[last] < best_weight) {
                    best_cut = cut;
                    best_weight = w[last];
                }
            } else {
                for (int j = 0; j < N; j++)
                    w[j] += weights[last][j];
                added[last] = true;
            }
        }
    }
    return make_pair(best_weight, best_cut);
}


int main() {
    int N, M;
    while(cin >> N >> M) {
        vvi g(N, vi(N, 0));
        rep(i, M) {
            int f,t,c;
            cin >> f >> t >> c;
            g[f][t] += c;
            g[t][f] += c;
        }

        cout << GetMinCut(g).first << endl;
    }

    return 0;
}
```


## POJ3155 Hard Life




