# Summary
- Requirement
- Writing article
- References

# Requirement
- Node.js
- Hexo

## Node.js
It is good to use one of the following tools.

- [riywo/ndenv](https://github.com/riywo/ndenv)
- [creationix/nvm](https://github.com/creationix/nvm)

## Hexo
```bash
$ npm install -g hexo
```

# Writing article
## Clone this repository
```bash
$ git clone git@github.com:unique-sister/hexo.git unique-sister
```

## Install dependent packages
```bash
$ cd unique-sister
$ npm install
```

## Create new entry
```bash
$ hexo new "<Article Title>"
```

## Header guide
```
tags:
- author
- contest
- language
```

e.g.)

```
tags:
- necojackarc
- TopCoder
- Python

```

## Run local server
```bash
$ hexo server
```

- [http://localhost:4000/](http://localhost:4000/)

## Push & Deploy
When master repository is updated, CircleCI is to deploy automatically.

# References
- [いまアツいJavaScript！ゼロから始めるNode.js入門〜5分で環境構築編〜](http://liginc.co.jp/web/programming/node-js/85318)
- [所要時間3分!? Github PagesとHEXOで爆速ブログ構築してみよう！](http://liginc.co.jp/web/programming/server/104594)
- [チームブログをGitHubとHexoではじめよう！](http://blog.otakumode.com/2014/08/08/Blogging-with-hexoio/)
