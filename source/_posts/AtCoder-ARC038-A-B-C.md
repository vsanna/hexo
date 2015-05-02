title: "AtCoder ARC038 A, B, C"
date: 2015-05-03 05:50:26
tags:
- necojackarc
- AtCoder
- Ruby
---

2015/05/02 開催の ARC038 に参戦しました。
1問目しか解けないという見事なボコられっぷりでした。

即日解説が行われるので、その解説を読んで勉強し B, C も解いてみました。

<iframe src="//www.slideshare.net/slideshow/embed_code/key/ezUDHKNsPLs8y0" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/chokudai/arc038" title="AtCoder Regular Contest 038 解説" target="_blank">AtCoder Regular Contest 038 解説</a> </strong> from <strong><a href="//www.slideshare.net/chokudai" target="_blank">chokudai</a></strong> </div>

# ポイント
今回は全てゲーム系の問題でした。解説によると、ゲーム系の問題は以下のいずれかで解けるそうです。

- 後ろから探索する
- grundy数を求める
- adhoc な必勝法を見つける

grundy 数については全く知らなかったので、

## grundy 数
grundy 数とは、

> ある状態から1回の遷移で行ける状態の grundy 数の集合に含まれない最小の非負整数

のことらしいです。ナンノコッチャ。

grundy 数を用いれば、Nim というゲームの勝敗判定を他のゲームにも適用可能にすることができるそうです。

詳細については、[grundy数を習得したかった](http://d.hatena.ne.jp/nanikaka/20120524/1337797626)という記事が非常に参考になりました。
また、その記事に載っていた WL-Algorithm というゲーム理論においてもっとも単純なアルゴリズムも今回は使用します。
つまりどういうことかというと、その記事はとても参考になるので是非読んでおきましょうってことです！

# A - カードと兄妹
両方が最善尽くす場合、お互いその時に残っている一番大きい数字を取ることになります。
つまり、カードを降順にソートし、奇数番目を拾っていけば先手が得られるカードとなります。
その合計を求めれば良いですね。楽勝！

```ruby
_n = gets
puts gets.chomp.split("\s").map(&:to_i).sort { |i, j| j <=> i}.select.with_index { |a, i| i % 2 == 0 }.reduce(:+)
```

Ruby は実行する順番に実装していけば良いので、とても直感的にコーディングができてよいです。

# B - マス目と駒
ここで死にました。

この問題から部分点があります。
問題を読んだとき、おそらく馬鹿正直に全探索すれば部分点のみで、一工夫加えたら満点がとれるんだろうな〜と察したのですが、どうやらその通りだったようです。

そして諦めて全探索 (のつもり) で実装したにも関わらず死にました。ダメダメでした。

この問題は、先ほど軽く言及した WL-Algorithm を用いることで簡単に解けます。
開放の種類としては「後ろから探索する」に当てはまるそうです。

## 考え方
各マスでの勝敗状態を後ろから探索していくことで、スタート地点の勝敗状態を導きます。

- 遷移先の状態に勝利状態しか存在しない場合、現在の状態は敗北状態となる
- つまり、現在の位置から移動可能な範囲の状態を再帰的に調べれば良い
- また、障害物や盤外を勝利状態とみなすと実装が楽になる
    - 周りが全て移動不可の地点 -> 敗北状態 -> 遷移先に勝利状態のみ存在

移動不可の地点を勝利状態と定義するのは、最初よくわかりませんでしたが、実際に探索を順番に追ってみたら理解できました。
いつか、こういう実装を楽にする技を思いつけるようになるのだろうか…アルゴリズマーへの道は遠い。

## 実装
馬鹿正直に実装した場合、一度調べた地点の状態も調べなおしてしまいます。
それは全く嬉しくなく、そもそも満点が取れる計算量じゃなくなってしまうので、一工夫します。

一度調べた地点はメモっておけば良いですね。メモ化再帰と言うらしいです！

```ruby
def judge(i, j)
  return true if H == i || W == j || $matrix[i][j] == '#'
  return $memo[i][j] unless $memo[i][j] == nil
  return true unless judge(i + 1, j)
  return true unless judge(i + 1, j + 1)
  return true unless judge(i, j + 1)
  $memo[i][j] = false
end

H, W = gets.chomp.split("\s").map(&:to_i)
$matrix = H.times.map { gets.chomp.split('')  }
$memo = H.times.map { W.times.map {} }
puts judge(0, 0) ? 'First' : 'Second'
```

無事通りました。`return`が並んでて超キモいです。

# C - 茶碗と豆
たどり着きもしなかった問題です。

ただ、B 問題と同じく、プレイヤーが交代で操作をし、常に状態の遷移先が少なくなっていきます。
競プロでのゲーム問題というのは、こういう問題が多いんじゃないかなとなんとなく思いました。

## 考え方
ここでは grundy 数を使います。使うと解けるそうです。凄いね。

- 各茶碗に豆が1つだけ入ってる場合の grundy 数を全ての茶碗で求める
- 全ての豆についての grundy 数の`xor`を求める

豆が n 個ある場合というのは、n 個の盤面 (茶碗の並び) にそれぞれ豆が1つだけ入ってるとみなすことができます。
そのため、まずは求めやすい豆が1つだけの grundy 数を 0 番目の茶碗から順に、全ての茶碗で求めていきます。
0 番目の茶碗は動かすことのできない敗北状態の茶碗で、grundy 数が確定していますので、そこを起点に順番に求めていくというわけです。
そして、それぞれの盤面の grundy 数を`xor`で重ね合わせれば、最終的な grundy 数を求めることができます。

なぜ`xor`でそれが可能なのかは、先ほどのブログが非常に参考になります。
理解したような、あんまり理解してないような、そんな感じです。
Nim に落とし込んでの説明だったら、なんとなくは可能な気がします。そんな理解度。

## 実装
各茶碗で、豆が1つだけ存在する場合の grundy 数を求めて、最後に`xor`で重ねます。
また、計算量を減らす工夫として、`A xor A = 0`という性質を利用しています。

茶碗に豆が n 個あった場合の grundy 数 (g とおく) は、`g xor g xor ... g`と、同じ値を n 回`xor`することになります。
つまり、n が偶数であった場合、最終的にその茶碗の grundy 数は0になります。
そこで、`a[i] = a_i % 2`とし、豆の数を0と1のみに圧縮しています。

```ruby
n = gets.chomp.to_i
g = [0]
a = [0]
(1..n-1).each do |i|
  c_i, a_i = gets.chomp.split("\s").map(&:to_i)
  g_tmp = g[i-c_i..i-1]
  g[i] = (0..g_tmp.max+1).select { |x| !g_tmp.include?(x) }.min
  a[i] = a_i % 2
end
puts (g.zip a).map { |x, y| x*y }.reduce(:^) != 0 ? 'First' : 'Second'
```

最終行ですが、本当は Haskell の zipWith 関数的なのを使いたかったのですが、 Ruby にはないため、`zip` + `map`で代用しています。
思いつきで書きましたが、割と綺麗に zipWith の挙動を再現できてるんじゃないかなと思います。

```ruby
(arr1.zip arr2).map { |x, y| hoge }
```

そして実は、この回答は実は満点ではありません。
得点としては、100/104になります。あと4点！！

現在、豆が1つの場合の各茶碗の grundy 数を求める処理を工夫なく書いていますが、ここを改善しなくては満点に届きません。
セグメントツリーというデータ構造を用いると良いそうです。。できれば早いうちにチャレンジしてみたいです。

セグメントツリーについては、以下の資料がわかりやすかったです。

<iframe src="//www.slideshare.net/slideshow/embed_code/key/8A8N1OximmsFW2" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/iwiwi/ss-3578491" title="プログラミングコンテストでのデータ構造" target="_blank">プログラミングコンテストでのデータ構造</a> </strong> from <strong><a href="//www.slideshare.net/iwiwi" target="_blank">Takuya Akiba</a></strong> </div>

蟻本の方のスライドです！

# 学んだこと
今回はざっくり以下のアルゴリズムについて学ぶことができました。

- WL-Algorithm
- メモ化再帰
- grundy 数

使いこなせるよう、しっかり脳に叩き込んでおきます！
