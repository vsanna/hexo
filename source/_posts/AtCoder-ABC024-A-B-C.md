title: "AtCoder ABC024 A, B, C"
date: 2015-05-24 02:43:12
tags:
- necojackarc
- AtCoder
- Ruby
---

AtCoder Beginner Contest 023 の A, B, C を解いてみました。
D 問題は数学ゲーだったようですが、まだ問題文すら読んでいません。

# 公式解説
<iframe src="//www.slideshare.net/slideshow/embed_code/key/MaCdSDNRXfyGsT" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/chokudai/abc024" title="AtCoder Beginner Contest 024 解説" target="_blank">AtCoder Beginner Contest 024 解説</a> </strong> from <strong><a href="//www.slideshare.net/chokudai" target="_blank">chokudai</a></strong> </div>

# A - 動物園
値段の合計を出して、合計人数によってはそこから割引を行えば良いです。

```ruby
A, B, C, K = gets.chomp.split("\s").map(&:to_i)
S, T = gets.chomp.split("\s").map(&:to_i)

total_member = S + T
total_cost = (A * S) + (B * T)

if total_member >= K
  puts total_cost - (C * total_member)
else
  puts total_cost
end
```

説明変数でわかりやすく！

# B - 自動ドア
公式解説よりシンプルなアルゴリズムで実装できたような気がします。

解説では「空いている時刻を調べる（部分点解法）」と「何秒後に閉まるかを調べる（満点解法）」と、時刻に注目したアルゴリズムでしたが、人が通過した時間のみに着目して解いてみました。

ドアが開いている時に次の人が来たかどうかという点のみに注目して解いています。

ドアが開いている時に来たら、来た時刻 - 前の人が来た時刻をドアが開いていた合計時間に加え、そうでなければ T (Tはドアが1回で開く時間) を加えるていきます。
最後の人は常に T 時間ほどドアを開けることになると思うので、あらかじめ合計に T を加えておきます。
説明が雑ですが、こうすることでドアが開いている合計時間を前の人の到着時刻を見るだけで導けます。

```ruby
N, T = gets.chomp.split("\s").map(&:to_i)

ans = T
before = gets.chomp.to_i
(N-1).times do
  current = gets.chomp.to_i
  ans += current - before < T ? current - before : T
  before = current
end

puts ans
```

# C - 民族大移動
綺麗に解く方法が思いつかなかったので、公式解説を参考にしました。
制約も緩いので、貪欲法でひとつずつ計算すれば間に合います。

```ruby
N, D, K = gets.chomp.split("\s").map(&:to_i)
L, R = D.times.map { |_| gets.chomp.split("\s").map(&:to_i) }.transpose
S, T = K.times.map { |_| gets.chomp.split("\s").map(&:to_i) }.transpose

K.times do |i|
  current, goal = S[i], T[i]
  D.times do |j|
    next if current < L[j] || R[j] < current
    if L[j] <= goal && goal <= R[j]
      puts j + 1 # answer
      break
    else
      current = (goal - L[j]).abs < (goal - R[j]).abs ? L[j] : R[j]
    end
  end
end
```

Ruby 超快適。
