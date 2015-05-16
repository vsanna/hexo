title: "TopCoder SRM 383 Div.2 Easy, Medium"
date: 2015-05-01 03:26:31
tags:
- necojackarc
- TopCoder
- Python
---

SRM 383 Div2 の Easy と Medium を Python で解いてみました。

# Easy. FloorLayout
行に`-`の塊が何個あるか、列に`|`の塊が何個あるかを数える問題です。

```
layout: {"----",
         "----",
         "----",
         "----"}
```

だと、4になり、

```
layout: {"|---",
         "|---",
         "|---",
         "|---"}
```

だと、5になります。

## 解法
行に対しては`|`で分割し、列に対しては`-`で分割し、何個の塊ができるかを数えれば良さそうです。

各行をセパレータで分割し、分割後の要素がいくつになるかを数える関数を作りました。
転置してやれば行に対しての操作を列に対して行えるため、ソースの重複が減って気持ちが良いです。

```python
class FloorLayout:
    def countBoards(self, layout):
        return self.countRowBoards(layout, '|') + \
            self.countRowBoards(map(lambda x: ''.join(x), map(list, zip(*layout))), '-')

    def countRowBoards(self, layout, sep):
        count = 0
        for row in layout:
            tmp =  row.split(sep)
            while '' in tmp:
                tmp.remove('')
            count += len(tmp)
        return count
```

# Medium. Planks
いくつかの長さが違う木が与えられ、それを切りそろえて売ります。同じ長さに切り揃えたモノしか売れません。また、木を切るのにもコストがかかります。コストと木の単価が与えられるので、売値を最大にするようにしたいです。

## 解法
綺麗に切り揃える方法は全く思いつきません。

与えられる木の数は最大50個、木の最大の長さは10000とのことなので、全通りの木の切りを試しても、計算回数は50 * 10000 = 500000となると思われます。10^6を切っているので、おそらく全探索可能です。

また、木を切らない方が良いパターンも存在します。その場合、切ることによりコストの方が高くなるため、切らずに売らないことにします。

```Python
class Planks:
    def makeSimilar(self, lengths, costPerCut, woodValue):
        MAX_LEN = 10000

        max_value = 0
        for cut_length in range(1, MAX_LEN + 1):
            value = 0
            for length in lengths:
                unit_count = length / cut_length
                cut_count = math.ceil(float(length) / cut_length) - 1
                tmp_value = woodValue * cut_length * unit_count - costPerCut * cut_count
                value = value + tmp_value if 0 < tmp_value else value
            max_value = value if max_value < value else max_value

        return max_value
```

入力値の制約から適切なアルゴリズムを推測できるようになりたい。
