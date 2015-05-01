# Summary
- Requirement
- Writing article

# Requirement
- Node.js
- Hexo

## Node.js
### Install nvm
```bash
$ git clone git://github.com/creationix/nvm.git ~/.nvm
$ source ~/.nvm/nvm.sh
```

### Install Node.js
```bash
$ nvm ls-remote
$ nvm install 0.12.2
```

### Set default
```bash
$ nvm alias default v0.12.2
```

### Config .bash_profile
```bash
if [[ -s ~/.nvm/nvm.sh ]];
 then source ~/.nvm/nvm.sh
fi
```

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
- content
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
When push, CircleCI is to deploy automatically.

```bash
$ git push origin master
```

# Reference
- [いまアツいJavaScript！ゼロから始めるNode.js入門〜5分で環境構築編〜](http://liginc.co.jp/web/programming/node-js/85318)
- [所要時間3分!? Github PagesとHEXOで爆速ブログ構築してみよう！](http://liginc.co.jp/web/programming/server/104594)
- [チームブログをGitHubとHexoではじめよう！](http://blog.otakumode.com/2014/08/08/Blogging-with-hexoio/)
