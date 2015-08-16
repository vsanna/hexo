title: "Ari-practice solve #3"
date: 2015-08-16 01:59:26
tags:
- esplo77
- C++
- 蟻本
---

# 蟻本練習問題を解いた (3)
探索&探索&探索。
POJは謎。

## POJ2718 - Smallest Difference
全探索で間にあう。
ただし、POJの（暗黙の）制約が厳しいので、定数倍高速化しないといけない。
以下の工夫を盛り込む。

- 順列を求めるのは1回だけにする
- 分割後の数字はなるべく近い方がいいので、桁数は（長さ/2）のみを試せば良い
- 配列の分割に新しいvectorを生成して……。という余裕をかますとTLE

問題文にテストケース数の上限くらいは書いておいて欲しい……。
入力もめんどくさいし、嫌な問題。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int L;
    cin >> L;
    cin.ignore();

    rep(testcase, L) {
        string str;
        getline(cin, str);

        vi a;
        for (int i = 0; i < str.size(); i += 2)
            a.push_back(str[i] - '0');

        int res = INF;

        do {
            int lefts = 0;
            int lefte = a.size() / 2;
            int rights = lefte;
            int righte = a.size();

            if (a[lefts] == 0 && lefte-lefts != 1)
                continue;
            if (a[rights] == 0 && righte-rights != 1)
                continue;

            int l = 0, r = 0;
            rep(i, lefte) {
                l *= 10;
                l += a[i];
            }
            REP(i, rights, righte) {
                r *= 10;
                r += a[i];
            }

            res = min(res, abs(l - r));
        } while (next_permutation(all(a)));

        cout << res << endl;
    }

    return 0;
}
```


## POJ3187 - Backward Digit Sums
またnext_permutation問題。
問題文を素直に実装したらACするはずだが、何故かWAをくらい続けた。
コンビネーションで書き直し、最初に見つかった答えで決め打ちしてAC。
どこかバグっていたのだろうか。POJではよくわからん……。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, final;
    cin >> N >> final;

    vi a(N);
    rep(i,N) a[i] = i+1;

    int comb[N][N];
    memset(comb, -1, sizeof(comb));
    rep(i,N)
        comb[i][0] = comb[i][i] = 1;
    REP(i,1,N) {
        REP(j,1,i) {
            comb[i][j] = comb[i-1][j-1] + comb[i-1][j];
        }
    }

    do {
        int sm = 0;
        rep(i,N) {
            sm += a[i] * comb[N-1][i];
        }

        if(sm == final) {
            rep(i,N) {
                if(i!=0)
                    cout << " " ;
                cout << a[i];
            }
            cout << endl;
            return 0;
        }
    } while(next_permutation(all(a)));

    return 0;
}
```

## POJ3050 - Hopscotch
実装するだけ。
今までで一番簡単かも。
……簡単なはずが、何故か出来上がった数字をstringで保持したらWA。
intにしたらAC。
やってることは何も変わらないのに……。
POJのC++コンパイラ、文字列の扱いにバグがあるんじゃないかと疑うレベル。

```C++
int a[5][5];
set<int> res;

int dr[] = {1,0,-1,0};
int dc[] = {0, 1, 0, -1};

void dfs(int r, int c, int now, int count) {
    rep(i,4) {
        int nr = r + dr[i];
        int nc = c + dc[i];

        if(nr < 0 || 5 <= nr || nc < 0 || 5 <= nc)
            continue;

        int next = now * 10 + a[nr][nc];
        if(count == 4)
            res.insert(next);
        else
            dfs(nr, nc, next, count+1);
    }
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    rep(i,5) rep(j,5) cin >> a[i][j];

    rep(r,5) {
        rep(c,5) {
            dfs(r,c,a[r][c], 0);
        }
    }

    cout << res.size() << endl;

    return 0;
}
```

## AOJ0525 - おせんべい
上手に焼けました。
実装するだけ。縦をひっくり返す箇所は、実際にひっくり返す必要はない。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int R, C;
    while(cin >> R >> C, (R||C)) {

        bool sen[R][C];
        memset(sen, 0, sizeof(sen));
        rep(r, R) {
            rep(c, C) {
                int b;
                cin >> b;
                if (b == 1)
                    sen[r][c] = true;
            }
        }

        int res = 0;
        rep(bits, 2 << R) {
            rep(i, R)
                if (((bits >> i) & 0x1) == 1)
                    rep(c, C) sen[i][c] ^= 0x1;

            int tmpRes = 0;
            rep(c, C) {
                int trueNum = 0;
                rep(r, R)
                    if (sen[r][c]) trueNum++;
                tmpRes += max(trueNum, R - trueNum);
            }
            res = max(res, tmpRes);

            rep(i, R)
                if (((bits >> i) & 0x1) == 1)
                    rep(c, C) sen[i][c] ^= 0x1;
        }

        cout << res << endl;
    }
    
    return 0;
}
```


## まとめ
ひたすら全探索した。
dfs、順列、ビット列挙くらいが使えれば、基礎はOKだと思う。
ここの問題はどちらかと言うと（POJのせいで）定数倍高速化ゲーになっていたような……。
