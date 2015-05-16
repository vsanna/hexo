title: "TopCoder SRM 659 Div.2 Medium"
date: 2015-05-16 20:40:00
tags:
- necojackarc
- TopCoder
- Python
---

先日行われた SRM 659 Div.2 に参戦しました。
Hard わかんねーって思って結果を見た結果、なんと正答率0%だったようです。

とりあえず、Medium について書きます。

# Medium. PublicTransit
R 行 C 列の格子があり、そこへセル同士をつなぐワープポイントを1つ設置できます。
移動距離が最も長くなるものを最小にするよう設置した時、その距離はいくつになるかという問題です。

なにやら複雑そうな感じがしますが、なんとなくで解けました。

なんとなく、ワープポイントは対象に置くのが良さそうな気がするので対象に置く場合を考えます。
対象に置くと、どうやら同じ大きさの2つのフィールドに分割されるような気がします。
そして、その分割したフィールド内での最大移動距離が、全体での最大移動距離になりそうです。

つ！ま！り！出来るだけ小さくフィールドを均等に二分割して、フィールド内での最大移動距離を出せば良い！

なんという根拠が怪しい理論。でも、なんとなくそんな感じがします。

出来るだけ小さく二分するには、行と列のうち長い方の中心でぶった切ってしまえば良いです。
そして、各フィールドでの最大移動距離はそれぞれの行数 + 列数 - 2で求めることができます。

書いてみます。

```Python
# -*- coding: utf-8 -*-
import math,string,itertools,fractions,heapq,collections,re,array,bisect

class PublicTransit:
    def minimumLongestDistance(self, R, C):
        bigger = max(R, C)
        smaller = min(R, C)

        return int(math.ceil(bigger/float(2))) + smaller - 2
```

なんと通りました。めでたしめでたし。
