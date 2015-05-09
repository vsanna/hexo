title: "AtCoder ABC023"
date: 2015-05-10 03:16:15
tags:
- necojackarc
- AtCoder
- Ruby
- Scala
---

AtCoder Beginner Contest 023 に参戦しました。
自力で解けたのは C の部分点までで、残りは解説を見てからの実装です。

問題文を解決可能な問題へ置き換える能力がまだまだ足りてないなというのが感想です。

# 公式解説
以下が公式の解説です。

<iframe src="//www.slideshare.net/slideshow/embed_code/key/c1Y1imi306Maah" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/chokudai/abc023" title="AtCoder Beginner Contest 023 解説" target="_blank">AtCoder Beginner Contest 023 解説</a> </strong> from <strong><a href="//www.slideshare.net/chokudai" target="_blank">chokudai</a></strong> </div>

AtCoder は公式が即解説を行ってくれるのが非常に良いです。

# A - 加算王
二桁の整数が与えられ、それの各位の和を出力する問題。
ワンライナーで行けます。

```ruby
puts gets.chomp.split('').map(&:to_i).reduce(:+)
```

# B - 手芸王
とある手順で文字列を作っていき、与えられた文字列ができるかどうかを調べる問題。
制約が非常に緩いので、手順通りに文字列を作っていけば十分間に合います。

```ruby
n = gets.chomp.to_i
s = gets.chomp

if s == 'b'
  puts 0
  exit
end

acc = 'b'
(1..50).each do |i|
  if i == 50
    puts -1
    exit
  end

  acc =
    case i % 3
    when 1
      'a' + acc + 'c'
    when 2
      'c' + acc + 'a'
    when 0
      'b' + acc + 'b'
    end

  if acc == s
    puts i
    exit
  end
end
```

この回答は正直汚いです。答えられればいいんだよ！感満載の回答です。
ループ条件を「与えられた文字列の長さ未満の間」にすると多分綺麗に書けます。

# C - 収集王
突然制約が厳しくなりました。ただ、問題自体は非常に単純です。
いかに計算量を減らす方法を考えるかが重要になる問題です。

工夫する力が足りない！思考の柔軟性が足りない！

```ruby
R, C, K = gets.chomp.split("\s").map(&:to_i)
N = gets.chomp.to_i

r, c = [[], []] # i番目の飴: r[i]行のc[i]列目
r_sum, c_sum = [Array.new(R, 0), Array.new(C, 0)] # i行目の合計, i列目の合計
N.times do |n|
  i, j = gets.chomp.split("\s").map(&:to_i)
  r[n], c[n] = [i-1, j-1]
  r_sum[r[n]] += 1
  c_sum[c[n]] += 1
end

r_num, c_num = [Array.new(N+1, 0), Array.new(N+1, 0)] # 飴がi個ある列の数, 行の数
R.times do |i|
  r_num[r_sum[i]] += 1
end
C.times do |i|
  c_num[c_sum[i]] += 1
end

count = (0..K).map do |i|
  r_num[i] * c_num[K-i]
end.reduce(:+)

N.times do |n|
  case r_sum[r[n]] + c_sum[c[n]]
  when K
    count -= 1
  when K + 1
    count += 1
  end
end

puts count
```

ほぼ公式解説のアルゴリズムです。

# D - 射撃王
柔軟に考えて解決可能な問題に再定義できるかが問われる問題です。
その辺できるようにもっと頑張っていきたいです。

今回は「得られる得点をできるだけ小さくする」というのを「X点以下の得点を取れるかどうか」という判定問題に変換し、取得可能な得点の最小値を求めることで解決可能になります。

The 二分探索問題といった感じです。

判定問題に落とし込み二分探索という手法は身体に染み込ませておきたいところです。
この辺りの勘は経験を積めば働くようになっていく……はず……！

```scala
object Main {
  def main(args: Array[String]): Unit = {
    val sc = new java.util.Scanner(System.in)
    val N = sc.nextInt()
    val H, S = new Array[Int](N)
    for (i <- 0 to N-1) {
      H(i) = sc.nextInt()
      S(i) = sc.nextInt()
    }

    var left: Long = 0
    var right: Long = Math.pow(10, 18).toLong
    while (right - left > 1) {
      val mid = (left + right) / 2
      if (isPossible(H, S, mid)) right = mid
      else left = mid
    }
    println(right)
  }

  def isPossible(H: Array[Int], S: Array[Int], limit: Long): Boolean = {
    val N = H.length
    val bucket = new Array[Int](N + 1)
    for (i <- 0 to N - 1) {
      if (H(i) > limit) return false
      val sec = List((limit - H(i)) / S(i), N.toLong).min.toInt
      bucket(sec) += 1
    }

    var sum = -1
    for (i <- 0 to N - 1) {
      sum += bucket(i)
      if (sum > i) return false
    }
    true
  }
}
```

突然の Scala ！

なんと、Ruby で実装したところ解説と同じアルゴリズムなのにもかかわらず TLE で落ちました。
Scala で実装したところ、問題なく AC ……やはり Ruby や Python で競プロは茨の道なのでしょうか。

# 感想
- 問題を別問題に置き換える思考は超基本で超大切
    - 判定問題化して二分探索を使いこなしたい！
- 積極的に Scala 使っていく
    - TopCoder は Scala 使えないからとりあえず Python
