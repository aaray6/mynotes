my notes(blog) by hexo

## install nodejs

```console
apt install nodejs
```

## install hexo

```console
npm install hexo-cli -g
```

## Download mynotes

```console
cd dev
git clone https://github.com/aaray6/mynotes.git
cd mynotes
npm install
npm install hexo-renderer-pug hexo-renderer-stylus hexo-deployer-git
```

## build & test

```console
hexo g
hexo s
```

浏览器打开http://localhost:4000
