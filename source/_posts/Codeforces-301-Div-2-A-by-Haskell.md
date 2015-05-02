title: "Codeforces #301 Div.2 A by Haskell"
date: 2015-05-02 12:06:37
tags:
- necojackarc
- Codeforces
- Haskell
---

業務のスキマ時間に Haskell で1問解いてみました。

# A. Combination Lock
n 個のディスクからなるロックの初期状態と解錠状態が与えられ、最短何回で解錠できるかを求める問題です。

解法としては、それぞれのディスクで最小の回数を求め、それの合計を求めれば良いです。
各ディスクでの最小は、`|初期番号 - 解錠番号|`と`10 - |初期番号 - 解錠番号|`、つまり順方向に回した場合と逆方向に回した場合の小さい方となります。

```haskell
import Data.Char

minMoves :: [Int] -> [Int] -> Int
minMoves xs ys = sum $ zipWith unitMinMoves xs ys
  where
  unitMinMoves x y = if i < j then i else j
    where
    i = abs $ x - y
    j = 10 - i

main :: IO ()
main = do
  _ <- fmap (read :: String -> Int) getLine
  origin <- fmap (map digitToInt) getLine
  answer <- fmap (map digitToInt) getLine
  print $ minMoves origin answer
```

こういう場合`zipWith`がいい仕事をしてくれますね。
もっと Haskell のエレガントさを見せつけていけるよう精進したいです。

# 番外編
会社のワンライナーマスターがターミナル (bash) からワンライナーで解いてくださいました。

```bash
$ cat input|tail -2|xargs echo|perl -ple "s//0/g"|perl -ple "s/0\s+0/+1/g"|bc|perl -ple "s/^1//"|perl -ple "s/(\d)(\d)/[sort {\$a <=> \$b} (abs(\$1-\$2),10-abs(\$1-\$2))]->[0] . \"+\" /ge"|perl -ple "s/\+$//"|bc
```

ヤバい。
