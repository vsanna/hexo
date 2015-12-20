title: "Topcoder SRM 663 Div.1 Easy"
date: 2015-12-20 21:53:01
tags:
- esplo77
- Topcoder
- C++11
---

# ABBADiv1
[問題文](https://community.topcoder.com/stat?c=problem_statement&pm=13922&rd=16512)

## 考え方
initialからtargetを作ろうとすると、2通りの操作があるので、最悪時は$2^{50}$ほど掛かってしまい、当然TLEする。
そこで、逆にtargetからinitialになるかを考えると、targetの部分集合を探せばいいので、状態数がかなり落ちる。
targetを削って出来る全ての集合を記録し、それらの中でinitialに一致するものがあるかを判定すればよい。

### 計算量
targetから削る際の操作は、

1. 後ろにAが付いている場合、最後を削る
2. 先頭にBが付いている場合、先頭を削って反転

という2種類になる。

ここで、ある文字列strがあるとし、上記2通りが両方適用できる（=2パターンに分岐する）場合は、B...Aとなる。
この場合、2の方法を適用すると、A... となり、二度と2が適用されることはない。
そのため、ほぼ$O(n)$に収まることが分かる。

### 備考
最初は、initialとtargetを両方正規化（上の2つの処理を貪欲に行う）し、一致するかを判定すればいいかと思ったが、Aを中途半端に削るケースがあるため出来なかった。

```C++
void normalize(string& str) {
    if(str.empty())
        return;

    cand.insert(str);

    string before = str;

    // Aを削るパターン
    if(!str.empty()) {
        if( str[str.size()-1] == 'A' ) {
            str = str.substr(0, str.size() - 1);
            normalize(str);
        }
    }

    // 元に戻す
    str = before;

    // 先頭にBがあれば、取り除いて反転
    if(!str.empty() && str[0]== 'B') {
        str = str.substr(1, str.size());
        reverse(all(str));
        normalize(str);
    }
}

string canObtain(string initial, string target) {
    cand.clear();
    normalize(target);

    if( cand.find(initial) != cand.end() )
        return  "Possible";
    else
        return "Impossible";
}
```
