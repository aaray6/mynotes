my notes(blog) by hexo

## install nodejs

```console
apt install nodejs
```

## install hexo

```console
npm install hexo-cli -g
```

## init hexo

```console
mkdir -p ~/dev
cd dev
hext init mynotes
```

## Download https://github.com/aaray6/mynotes.git

```console
cd /tmp
git clone https://github.com/aaray6/mynotes.git
cp -R /tmp/mynotes/* ~/dev/mynotes/
cd ~/mynotes
npm install hexo-renderer-pug hexo-renderer-stylus
```

## build & test

```console
cd ~/mynotes
hexo g
hexo s
```

http://localhost:4000

