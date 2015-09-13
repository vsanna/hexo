title: "TopCoder SRM 667 Div.1 Easy 250"
date: 2015-09-14 00:19:17
tags:
- esplo77
- Topcoder
- C++11
---

# 250 OrderOfOperations
やられた。

## 問題
[問題文](http://community.topcoder.com/stat?c=problem_statement&pm=13987&rd=16547)

複数のビット列の順番を並び替え、立っていないビットを立てるコストを最小にする問題。
コストは、立っていないビット数の二乗。

## 貪欲な考え方
制約を見ると非常に小さいため、色々な方法がありそうに見える。
本番中に考えたのは、「新しく上げるビット数が少ないものから貪欲に並べる」という方法。

一度立てたビットは、他の候補の対応するビットを下ろす。
オーダーは、1ループでソートに$O(n \cdot logn)$、ビット関連で$O(m \cdot n)$なので、およそ$O(n^2 \cdot m)$。

反例が見つからなかったため、これで投げてみた。
速攻で撃墜された。

ダメな入力は以下。
```text
"000011"
"001100"
"110000"
"111000"
"001110"
"000111"
```

このような場合、"110000"を最初に処理しないと、トータルのコストが最小にならない。
気づかんかった。


## 正しい考え方
貪欲には解けないので、DPで解く。

dp[i] := iにするために必要な最小コスト
と置く。
iを0 -> (1<<m)で回し、そのそれぞれで各候補を適用するとDAGになるため、綺麗にDPで解ける。
立っているビット数は、GCCなら__builtin_popcount(x)だと簡単に求められる。

```C++
const int MAX_DP = (1<<21) + 10;
int dp[MAX_DP];

class OrderOfOperations {
    public:

    const int INF = 1<<29;

    int minTime(vector<string> s) {
        int n=s.size();
        int m=s[0].size();

        // dp[i] := i にするために必要な最小コスト
        // ループは dp[i] -> dp[i|j] とするので、DAGになる
        fill(dp, dp+MAX_DP, INF);
        dp[0] = 0;

        vi bits(n);
        rep(i,n) {
            int t = 0;
            rep(j, m) {
                t <<= 1;
                t += s[i][j] - '0';
            }
            bits[i] = t;

            int counts = __builtin_popcount(bits[i]);
            dp[bits[i]] = counts*counts;
        }

        rep(j, (1<<m)) {
            tr(it, bits) {
                int apply = *it;
                int diff = __builtin_popcount(j|apply) - __builtin_popcount(j);
                dp[j | apply] = min(dp[j | apply], dp[j] + diff*diff);
            }
        }

        int fin = 0;
        rep(i,n) fin |= bits[i];
        return dp[fin];
    }
};
```

## まとめ
コーナーケースに気づいていれば解けたと思うので悔しい。
ここ最近1問も解けていないので、そろそろ解きたい。
