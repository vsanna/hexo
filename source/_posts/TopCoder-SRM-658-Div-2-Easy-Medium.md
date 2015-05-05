title: "TopCoder SRM 658 Div.2 Easy, Medium"
date: 2015-05-06 02:51:38
tags:
- necojackarc
- TopCoder
- Python
---

本日 (5/6 0:00 JST) 開催された TopCoder SRM 658 Div.2 に参戦しました。
実は初参戦でした。Arena の使い方がよくわからず死ぬかと思った。

アホなミスで Medium を落として Easy のみシステムテストクリアという散々な結果でした。
Medium はミス直したら即通りました。アルゴリズムに問題はなかった模様。悲しい。
これから少しずつ頑張っていきます……！！

# Easy. InfiniteString
一切工夫してない実装で無事通過。

## 問題
受け取った文字列を無限回繰り返す関数 f(x) があります。
与えられた2つの文字列 s, t (長さ1-50) を f(x) に適用した時、同じ文字列になるかどうか調べなさい。

## 考え方
同じ長さになるまで繰り返してみて、その文字列が同じかどうかを調べたら良い。
おそらく、最小公倍数になるまで、それぞれ足し続けることになる。
最小公倍数は多く見積もっても $50 \times 50 = 2500$ 未満だと思うので大丈夫。

## 実装
同じ長さになるまで、短い方に元の文字列を足し続ける。
最後に、文字列が等しいかどうかで返り値を決定する。

```python
class InfiniteString:
    def equal(self, s, t):
        (sx, tx) = (s, t)
        while len(sx) != len(tx):
            if len(sx) < len(tx):
                sx += s
            else:
                tx += t
        return "Equal" if sx == tx else "Not equal"
```

# Medium. MutaliskEasy
本番ではうっかりミスが2箇所もあってシステムテストで死にました。
深い考察なくヒューリスティックな実装をしましたが、通ったので良しとしましょう。

## 問題
敵が1-3体現れ、それぞれの HP が与えられます。
1ターンの攻撃で3回攻撃でき、各攻撃で9, 3, 1ポイント削ることができます。
ただし、同じ敵への攻撃はできません。
敵を倒す最小攻撃回数を求めてください。

## 考え方
3パターンしかないので場合分けして考えてみます。

### 敵が1体
毎回9ポイント与えるのがどう考えても最短。
敵の HP / 9 を切り上げた値が答えになるはず。

### 敵が2体
HP が多い方に9ポイントのダメージを与え続ければ良い気がする。
本番では全パターン調べたけど、少ない方に多くダメージを与えた方がいい場合がよくわからない。

### 敵が3体
一番 HP が多い敵に9ポイントのダメージを与えて、その他2体については2パターンとも考慮する。
最短のパターンが見つかれば計算打ち切り。幅優先探索っぽい何かな気がする。
全部調べたら余裕で計算が終わらなかったので、ヒューリスティックに探索範囲を絞ってみた。
なんでこれで大丈夫なのかは正直よくわかってないけど、何故かシステムテストは通る。

## 実装
`queue`と`tmp_queue`という配列を作って、そこに次に調べる状態を詰めてます。
同じ状態が現れたらスキップするというロジックも入れようとしたけど、なくても通ったので割愛。


```python
class MutaliskEasy:
    def minimalAttacks(self, x):
        x = list(x)

        if len(x) == 1:
            return int(math.ceil(float(x[0]) / 9))

        x.sort()
        count = 0

        if len(x) == 2:
            while True:
                count += 1
                x = self.nextState(x, [3, 9])
                if sum(x) <= 0:
                    return count

        else:
            tmp_queue = [x]
            while True:
                count += 1
                queue = tmp_queue
                tmp_queue = []
                for hp in queue:
                    for attack in [[3, 1, 9], [1, 3, 9]]:
                        tmp = self.nextState(hp, attack)
                        if sum(tmp) <= 0:
                            return count
                        else:
                            tmp_queue.append(tmp)

        return 0

    def nextState(self, hp, attack):
        tmp = map(lambda x, y: x - y if x - y > 0 else 0, hp, attack)
        tmp.sort()
        return tmp
```

# 感想
初参戦の TopCoder SRM 楽しかったです！可能な限り参加していきたい！
[TopCoder部のカレンダー](https://topcoder.g.hatena.ne.jp/calendar)を Google カレンダーに追加して準備ばっちし。
これでいろんなコンテストの情報が簡単にチェック可能！

あと、同じルームで Python 使ってる人は誰もいませんでした。
パッと見た感じ、C++ と Java ユーザだけな感じ。
C++ は未経験なのでパッと読むのが辛い。C なら多分わかるはず。
C++ と Java の読み書き練習した方が良い気がしてきた。
速度的にも C++ か Java がやっぱり良さそう。

やはり C++ か Java なのか……。
