# 概要
- Requirement 
- Writing article

# Requirement
- Node.js
- Hexo

## Node.js
- [いまアツいJavaScript！ゼロから始めるNode.js入門〜5分で環境構築編〜](http://liginc.co.jp/web/programming/node-js/85318)

### Install nvm
```bash
$ git clone git://github.com/creationix/nvm.git ~/.nvm
$ source ~/.nvm/nvm.sh
```

### Install Node.js
```bash
$ nvm ls-remote
$ nvm install 0.xx.xx
```

### Set default
```bash
$ nvm alias default v0.10.26
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
$ npm install hexo-deployer-git --save
```

# Writing article
## Clone this repository
```bash
$ git clone git@github.com:unique-sister/hexo.git unique-sister
```

## Create new entry
```bash
$ hexo new "<Article Title>"
```

## Run local server
```bash
$ hexo server
```

- [http://localhost:4000/](http://localhost:4000/)

## Deploy
```bash
$ hexo clean && hexo deploy -g
```
