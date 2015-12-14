title: "Codeforces #313 Div.2 D"
date: 2015-12-15 01:00:41
tags:
- esplo77
- Codeforces
- C++11
---

# D. Equivalent Strings

[問題文](http://codeforces.com/contest/560/problem/D)

## 考え方
愚直に実装すると解けはするが、TLEしそう。
D問題だし、ちゃんとTLEするのだろう。

「2つの要素を半分に分け、入れ替えたものが等しくても同じ」とみなすということは、制限の強いアナグラムの問題と考えることが出来る。
アナグラムの一致判定といえば、ソートして一致するか比較。
というわけで、再帰的に分割し、小さい方を手前にくっつける処理を行うことで一致するか判定することが出来る。

サンプル1を例に取ると、
aaba -> (aa),(ba) -> (a,a),(b,a) -> (aa),(ab) -> aaab
abaa -> (ab),(aa) -> (a,b),(a,a) -> (ab),(aa) -> aaab
となり一致する。

サンプル2では、
aabb -> (aa),(bb) -> (a,a),(b,b) -> (aa),(bb) -> aabb
abab -> (ab),(ab) -> (a,b),(a,b) -> (ab),(ab) -> abab
となり一致しない（少し分かりにくい例……）。

```C++
string solve(string s) {
    if (s.size() % 2 == 1)
        return s;
    string mini1 = solve(s.substr(0, s.size() / 2));
    string mini2 = solve(s.substr(s.size() / 2, s.size()));
    return min(mini1, mini2) + max(mini1, mini2);
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    string str1, str2;
    cin >> str1 >> str2;

    if (str1.size() % 2 == 1) {
        if (str1 == str2)
            cout << "YES" << endl;
        else
            cout << "NO" << endl;
    }
    else {
        if (solve(str1) == solve(str2))
            cout << "YES" << endl;
        else
            cout << "NO" << endl;
    }

    return 0;
}
```

## まとめ
本番では221/3023人が解いたようだ。