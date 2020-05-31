---
title: Hexo增加站内搜索功能以及随后遇到的问题
date: 2020-05-31 19:48:30
tags:
---

![hexo-generator-search](/myimages/hexo-add-search01.png "local search")

本来想增加一个站内搜索，以便以后找文章用，结果...越来越麻烦。

看介绍，我现在用的这个theme hexo-theme-melody就可以支持，本来很简单。

[Local search](https://molunerfinn.com/hexo-theme-melody-doc/third-party-support.html#local-search)

```console
You should install hexo-generator-search. Follow its doc to setup. Only supporting the xml file.

Set the melody.yml

local_search:
  enable: true # or false
  labels:
    input_placeholder: Search for Posts
    hits_empty: "We didn't find any results for the search: ${query}" # if there are no result
```

## 第一步，需要安装hexo-generator-search

跟着[这个文档](https://github.com/wzpan/hexo-generator-search)做就行。

```console
Install
$ npm install hexo-generator-search --save
```

结果

```console
$ npm install hexo-generator-search --save

Command 'npm' not found, but can be installed with:

sudo apt install npm
```

我的npm不见了，好吧，按照提示装吧。

## 安装npm

```console
$ sudo apt install npm
[sudo] xxxx 的密码： 
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
有一些软件包无法被安装。如果您用的是 unstable 发行版，这也许是
因为系统无法达到您要求的状态造成的。该版本中可能会有一些您需要的软件
包尚未被创建或是它们已被从新到(Incoming)目录移出。
下列信息可能会对解决问题有所帮助：

下列软件包有未满足的依赖关系：
 npm : 依赖: node-gyp (>= 0.10.9) 但是它将不会被安装
N: 忽略‘d-apt.list.bak1’(于目录‘/etc/apt/sources.list.d/’)，鉴于它的文件扩展名无效
N: 忽略‘fcitx-team-ubuntu-nightly-bionic.list.bak1’(于目录‘/etc/apt/sources.list.d/’)，鉴于它的文件扩展名无效
E: 无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系。
```

嗯，我的node-gyp (>= 0.10.9)没有，并且装不了。

## 修复node-gyp (>= 0.10.9)依赖不满足导致安装不了npm的问题

Google一下，找到[这个文章](https://askubuntu.com/questions/1057737/ubuntu-18-04-lts-server-npm-depends-node-gyp-0-10-9-but-it-is-not-going)

按照这里说的，只要

```console
Ubuntu Server 18.04 Node.js and npm install

sudo apt remove --purge nodejs npm
sudo apt clean
sudo apt autoclean
sudo apt install -f
sudo apt autoremove

Find the latest version at https://github.com/nodesource/distributions#debinstall Latest version 10.x now

sudo apt install curl
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
sudo apt-get update && sudo apt-get install yarn

npm -version
6.1.0
nodejs -v
v10.7.0
```

于是，按照以上步骤，先卸载node,再找到最新的node版本，然后安装node和npm,顺便把yarn也装上了。yarn是可以代替npm的一个包管理工具。

```console
$ curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -

## Installing the NodeSource Node.js 14.x repo...


## Populating apt-get cache...

+ apt-get update
命中:1 http://repo.steampowered.com/steam precise InRelease                   
获取:2 http://archive.ubuntukylin.com:10006/ubuntukylin xenial InRelease [18.1 kB]
命中:3 https://packages.microsoft.com/repos/vscode stable InRelease                        
命中:4 http://cn.archive.ubuntu.com/ubuntu bionic InRelease                                
获取:5 http://cn.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
获取:6 http://cn.archive.ubuntu.com/ubuntu bionic-updates/main amd64 DEP-11 Metadata [305 kB]
命中:7 http://download.virtualbox.org/virtualbox/debian bionic InRelease                        
获取:8 http://cn.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 DEP-11 Metadata [273 kB]                                          
获取:9 http://cn.archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 DEP-11 Metadata [2,468 B]                                       
命中:10 http://ppa.launchpad.net/linrunner/tlp/ubuntu bionic InRelease                                                                     
获取:11 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]                                                              
获取:12 http://security.ubuntu.com/ubuntu bionic-security/main amd64 DEP-11 Metadata [42.6 kB]                                             
获取:13 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 DEP-11 Metadata [42.1 kB]                                         
获取:14 http://security.ubuntu.com/ubuntu bionic-security/multiverse amd64 DEP-11 Metadata [2,464 B]                                       
已下载 863 kB，耗时 13秒 (68.8 kB/s)                                                                                                       
正在读取软件包列表... 完成
N: 鉴于仓库 'https://packages.microsoft.com/repos/vscode stable InRelease' 不支持 'i386' 体系结构，跳过配置文件 'main/binary-i386/Packages' 的获取。
N: 鉴于仓库 'http://download.virtualbox.org/virtualbox/debian bionic InRelease' 不支持 'i386' 体系结构，跳过配置文件 'contrib/binary-i386/Packages' 的获取。

## Confirming "bionic" is supported...

+ curl -sLf -o /dev/null 'https://deb.nodesource.com/node_14.x/dists/bionic/Release'

## Adding the NodeSource signing key to your keyring...

+ curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add -
OK

## Creating apt sources list file for the NodeSource Node.js 14.x repo...

+ echo 'deb https://deb.nodesource.com/node_14.x bionic main' > /etc/apt/sources.list.d/nodesource.list
+ echo 'deb-src https://deb.nodesource.com/node_14.x bionic main' >> /etc/apt/sources.list.d/nodesource.list

## Running `apt-get update` for you...

+ apt-get update
命中:1 http://download.virtualbox.org/virtualbox/debian bionic InRelease
命中:2 http://archive.ubuntukylin.com:10006/ubuntukylin xenial InRelease                                                                   
命中:3 https://packages.microsoft.com/repos/vscode stable InRelease                                                                        
命中:4 http://cn.archive.ubuntu.com/ubuntu bionic InRelease                                                                                
命中:5 http://security.ubuntu.com/ubuntu bionic-security InRelease                                                                         
命中:6 http://ppa.launchpad.net/linrunner/tlp/ubuntu bionic InRelease                                                                      
命中:7 http://repo.steampowered.com/steam precise InRelease                                                             
命中:8 http://cn.archive.ubuntu.com/ubuntu bionic-updates InRelease                                                     
获取:9 https://deb.nodesource.com/node_14.x bionic InRelease [4,584 B]                             
获取:10 https://deb.nodesource.com/node_14.x bionic/main amd64 Packages [766 B]
已下载 5,350 B，耗时 4秒 (1,389 B/s)
正在读取软件包列表... 完成
N: 鉴于仓库 'http://download.virtualbox.org/virtualbox/debian bionic InRelease' 不支持 'i386' 体系结构，跳过配置文件 'contrib/binary-i386/Packages' 的获取。
N: 鉴于仓库 'https://packages.microsoft.com/repos/vscode stable InRelease' 不支持 'i386' 体系结构，跳过配置文件 'main/binary-i386/Packages' 的获取。

## Run `sudo apt-get install -y nodejs` to install Node.js 14.x and npm
## You may also need development tools to build native addons:
     sudo apt-get install gcc g++ make
## To install the Yarn package manager, run:
     curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
     echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
     sudo apt-get update && sudo apt-get install yarn


$ sudo apt-get install -y nodejs
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
下列【新】软件包将被安装：
  nodejs
升级了 0 个软件包，新安装了 1 个软件包，要卸载 0 个软件包，有 28 个软件包未被升级。
需要下载 24.3 MB 的归档。
解压缩后会消耗 116 MB 的额外空间。
获取:1 https://deb.nodesource.com/node_14.x bionic/main amd64 nodejs amd64 14.3.0-1nodesource1 [24.3 MB]
已下载 24.3 MB，耗时 10秒 (2,375 kB/s)                                                                                                     
正在选中未选择的软件包 nodejs。
(正在读取数据库 ... 系统当前共安装有 327973 个文件和目录。)
正准备解包 .../nodejs_14.3.0-1nodesource1_amd64.deb  ...
正在解包 nodejs (14.3.0-1nodesource1) ...
正在设置 nodejs (14.3.0-1nodesource1) ...
正在处理用于 man-db (2.8.3-2ubuntu0.1) 的触发器 ...
quxr@quxr-ThinkPad-X201:/$ curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
OK
quxr@quxr-ThinkPad-X201:/$      echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
deb https://dl.yarnpkg.com/debian/ stable main
quxr@quxr-ThinkPad-X201:/$      sudo apt-get update && sudo apt-get install yarn
命中:1 http://archive.ubuntukylin.com:10006/ubuntukylin xenial InRelease       
命中:2 https://packages.microsoft.com/repos/vscode stable InRelease                                                              
命中:3 http://download.virtualbox.org/virtualbox/debian bionic InRelease                                                                   
命中:4 http://repo.steampowered.com/steam precise InRelease                                                          
获取:5 https://dl.yarnpkg.com/debian stable InRelease [17.1 kB]                                                      
命中:6 https://deb.nodesource.com/node_14.x bionic InRelease
命中:7 http://cn.archive.ubuntu.com/ubuntu bionic InRelease                      
获取:8 https://dl.yarnpkg.com/debian stable/main amd64 Packages [9,953 B]        
命中:9 http://cn.archive.ubuntu.com/ubuntu bionic-updates InRelease                                                                      
命中:10 http://ppa.launchpad.net/linrunner/tlp/ubuntu bionic InRelease                                                                   
获取:11 https://dl.yarnpkg.com/debian stable/main all Packages [9,953 B]
获取:12 https://dl.yarnpkg.com/debian stable/main i386 Packages [9,953 B]  
命中:13 http://security.ubuntu.com/ubuntu bionic-security InRelease
已下载 47.0 kB，耗时 6秒 (7,473 B/s)                                                                                                       
正在读取软件包列表... 完成
N: 鉴于仓库 'https://packages.microsoft.com/repos/vscode stable InRelease' 不支持 'i386' 体系结构，跳过配置文件 'main/binary-i386/Packages' 的获取。
N: 鉴于仓库 'http://download.virtualbox.org/virtualbox/debian bionic InRelease' 不支持 'i386' 体系结构，跳过配置文件 'contrib/binary-i386/Packages' 的获取。
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
下列【新】软件包将被安装：
  yarn
升级了 0 个软件包，新安装了 1 个软件包，要卸载 0 个软件包，有 28 个软件包未被升级。
需要下载 891 kB 的归档。
解压缩后会消耗 5,407 kB 的额外空间。
获取:1 https://dl.yarnpkg.com/debian stable/main amd64 yarn all 1.22.4-1 [891 kB]
已下载 891 kB，耗时 4秒 (202 kB/s)
正在选中未选择的软件包 yarn。
(正在读取数据库 ... 系统当前共安装有 332775 个文件和目录。)
正准备解包 .../archives/yarn_1.22.4-1_all.deb  ...
正在解包 yarn (1.22.4-1) ...
正在设置 yarn (1.22.4-1) ...
quxr@quxr-ThinkPad-X201:/$ which node
/usr/bin/node
quxr@quxr-ThinkPad-X201:/$ which npm
/usr/bin/npm
quxr@quxr-ThinkPad-X201:/$ which yarn
/usr/bin/yarn
quxr@quxr-ThinkPad-X201:/$ nodejs -v
v14.3.0
quxr@quxr-ThinkPad-X201:/$ node -v
v14.3.0
```

## 终于弄好了环境，开始回到第一步安装hexo-generator-search

原来给的npm install命令

```console
Install
$ npm install hexo-generator-search --save
```

换成yarn试试，文档说

Yarn和npm命令对比

npm install === yarn 
npm install taco --save === yarn add taco
npm uninstall taco --save === yarn remove taco
npm install taco --save-dev === yarn add taco --dev
npm update --save === yarn upgrade

那原来的命令就变成

```console
$ yarn add hexo-generator-search
yarn add v1.22.4
info No lockfile found.
[1/4] Resolving packages...
warning hexo > hexo-fs > chokidar@2.1.8: Chokidar 2 will break on node v14+. Upgrade to chokidar 3 with 15x less dependencies.
warning hexo > hexo-fs > chokidar > fsevents@1.2.13: fsevents 1 will break on node v14+ and could be using insecure binaries. Upgrade to fsevents 2.
warning hexo > warehouse > cuid > core-js@1.2.7: core-js@<3 is no longer maintained and not recommended for usage due to the number of issues. Please, upgrade your dependencies to the actual version of core-js@3.
warning hexo > hexo-fs > chokidar > braces > snapdragon > source-map-resolve > resolve-url@0.2.1: https://github.com/lydell/resolve-url#deprecated
warning hexo > hexo-fs > chokidar > braces > snapdragon > source-map-resolve > urix@0.1.0: Please see https://github.com/lydell/urix#deprecated
warning hexo-deployer-git > hexo-fs > chokidar@1.7.0: Chokidar 2 will break on node v14+. Upgrade to chokidar 3 with 15x less dependencies.
warning hexo-deployer-git > hexo-fs > chokidar > fsevents@1.2.13: fsevents 1 will break on node v14+ and could be using insecure binaries. Upgrade to fsevents 2.
warning hexo-deployer-git > swig@1.4.2: This package is no longer maintained
warning hexo-deployer-git > babel-eslint > babel-traverse > babel-runtime > core-js@2.6.11: core-js@<3 is no longer maintained and not recommended for usage due to the number of issues. Please, upgrade your dependencies to the actual version of core-js@3.
warning hexo-renderer-ejs > ejs@1.0.0: Critical security bugs fixed in 2.5.5
warning hexo-renderer-jade@0.5.0: hexo-renderer-jade has been deprecated. Please install hexo-renderer-pug and rename all *.jade files to *.pug.
warning hexo-renderer-jade > jade@1.11.0: Jade has been renamed to pug, please install the latest version of pug instead of jade
warning hexo-renderer-jade > jade > constantinople@3.0.2: Please update to at least constantinople 3.1.1
warning hexo-renderer-jade > jade > transformers@2.1.0: Deprecated, use jstransformer
warning hexo-renderer-sass > node-sass > request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
warning hexo-renderer-sass > node-sass > node-gyp > request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
warning hexo-renderer-stylus > stylus > css-parse > css > urix@0.1.0: Please see https://github.com/lydell/urix#deprecated
[2/4] Fetching packages...
info fsevents@1.2.13: The platform "linux" is incompatible with this module.
info "fsevents@1.2.13" is an optional dependency and failed compatibility check. Excluding it from installation.
info fsevents@2.1.3: The platform "linux" is incompatible with this module.
info "fsevents@2.1.3" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 357 new dependencies.
info Direct dependencies
├─ hexo-deployer-git@0.3.1
├─ hexo-generator-archive@0.1.5
├─ hexo-generator-category@0.1.3
├─ hexo-generator-index@0.2.1
├─ hexo-generator-search@2.4.0
├─ hexo-generator-tag@0.2.0
├─ hexo-renderer-ejs@0.2.0
├─ hexo-renderer-jade@0.5.0
├─ hexo-renderer-marked@0.2.11
├─ hexo-renderer-sass@0.3.2
├─ hexo-renderer-stylus@0.3.3
├─ hexo-server@0.2.2
└─ hexo@3.9.0
info All dependencies
├─ @types/babel-types@7.0.7
├─ @types/babylon@6.16.5
├─ a-sync-waterfall@1.0.1
├─ accepts@1.3.7
├─ acorn-globals@1.0.9
├─ ajv@6.12.2
├─ align-text@0.1.4
├─ ansi-styles@3.2.1
├─ anymatch@2.0.0
├─ aproba@1.2.0
├─ archy@1.0.0
├─ are-we-there-yet@1.1.5
├─ argparse@1.0.10
├─ arr-flatten@1.1.0
├─ array-find-index@1.0.2
├─ asap@2.0.6
├─ asn1@0.2.4
├─ assign-symbols@1.0.0
├─ async-each@1.0.3
├─ async-foreach@0.1.3
├─ asynckit@0.4.0
├─ atob@2.1.2
├─ aws-sign2@0.7.0
├─ aws4@1.10.0
├─ babel-code-frame@6.26.0
├─ babel-eslint@7.2.3
├─ babel-messages@6.23.0
├─ babel-runtime@6.26.0
├─ babel-traverse@6.26.0
├─ balanced-match@1.0.0
├─ base@0.11.2
├─ basic-auth@2.0.1
├─ bcrypt-pbkdf@1.0.2
├─ binary-extensions@1.13.1
├─ block-stream@0.0.9
├─ bluebird@3.7.2
├─ brace-expansion@1.1.11
├─ braces@2.3.2
├─ browser-fingerprint@0.0.1
├─ bytes@3.0.0
├─ cache-base@1.0.1
├─ camel-case@3.0.0
├─ camelcase-keys@2.1.0
├─ caseless@0.12.0
├─ center-align@0.1.3
├─ character-parser@1.2.1
├─ cheerio@0.22.0
├─ chokidar@2.1.8
├─ class-utils@0.3.6
├─ clean-css@3.4.28
├─ cliui@2.1.0
├─ code-point-at@1.1.0
├─ collection-visit@1.0.0
├─ color-convert@1.9.3
├─ color-name@1.1.3
├─ combined-stream@1.0.8
├─ command-exists@1.2.9
├─ commander@3.0.2
├─ compressible@2.0.18
├─ compression@1.7.4
├─ concat-map@0.0.1
├─ connect@3.7.0
├─ console-control-strings@1.1.0
├─ copy-descriptor@0.1.1
├─ core-js@1.2.7
├─ core-util-is@1.0.2
├─ cross-spawn@3.0.1
├─ css-parse@2.0.0
├─ css-select@1.2.0
├─ css-stringify@1.0.5
├─ css-what@2.1.3
├─ css@2.2.4
├─ cuid@1.3.8
├─ currently-unhandled@0.4.1
├─ dashdash@1.14.1
├─ decamelize@1.2.0
├─ decode-uri-component@0.2.0
├─ delayed-stream@1.0.0
├─ delegates@1.0.0
├─ destroy@1.0.4
├─ doctypes@1.1.0
├─ domhandler@2.4.2
├─ domutils@1.5.1
├─ ecc-jsbn@0.1.2
├─ ee-first@1.1.1
├─ ejs@1.0.0
├─ emoji-regex@7.0.3
├─ error-ex@1.3.2
├─ esprima@4.0.1
├─ etag@1.8.1
├─ expand-brackets@2.1.4
├─ expand-range@1.8.2
├─ extend@3.0.2
├─ extglob@2.0.4
├─ extsprintf@1.3.0
├─ fast-deep-equal@3.1.1
├─ fast-json-stable-stringify@2.1.0
├─ filename-regex@2.0.1
├─ fill-range@4.0.0
├─ finalhandler@1.1.2
├─ find-up@1.1.2
├─ for-in@1.0.2
├─ for-own@0.1.5
├─ forever-agent@0.6.1
├─ form-data@2.3.3
├─ fresh@0.5.2
├─ fstream@1.0.12
├─ function-bind@1.1.1
├─ gauge@2.7.4
├─ gaze@1.1.3
├─ get-caller-file@2.0.5
├─ getpass@0.1.7
├─ glob-base@0.3.0
├─ globals@9.18.0
├─ globule@1.3.1
├─ graceful-readlink@1.0.1
├─ har-schema@2.0.0
├─ har-validator@5.1.3
├─ has-ansi@2.0.0
├─ has-flag@3.0.0
├─ has-unicode@2.0.1
├─ has-value@1.0.0
├─ has@1.0.3
├─ hexo-bunyan@1.0.0
├─ hexo-cli@2.0.0
├─ hexo-deployer-git@0.3.1
├─ hexo-front-matter@0.2.3
├─ hexo-fs@1.0.2
├─ hexo-generator-archive@0.1.5
├─ hexo-generator-category@0.1.3
├─ hexo-generator-index@0.2.1
├─ hexo-generator-search@2.4.0
├─ hexo-generator-tag@0.2.0
├─ hexo-i18n@0.2.1
├─ hexo-renderer-ejs@0.2.0
├─ hexo-renderer-jade@0.5.0
├─ hexo-renderer-marked@0.2.11
├─ hexo-renderer-sass@0.3.2
├─ hexo-renderer-stylus@0.3.3
├─ hexo-server@0.2.2
├─ hexo@3.9.0
├─ highlight.js@9.18.1
├─ hosted-git-info@2.8.8
├─ html-entities@1.3.1
├─ htmlparser2@3.10.1
├─ http-errors@1.7.3
├─ http-signature@1.2.0
├─ in-publish@2.0.1
├─ indent-string@2.1.0
├─ inherits@2.0.4
├─ invariant@2.2.4
├─ is-accessor-descriptor@1.0.0
├─ is-arrayish@0.2.1
├─ is-data-descriptor@1.0.0
├─ is-descriptor@1.0.2
├─ is-dotfile@1.0.3
├─ is-equal-shallow@0.1.3
├─ is-expression@3.0.0
├─ is-finite@1.1.0
├─ is-plain-object@2.0.4
├─ is-posix-bracket@0.1.1
├─ is-primitive@2.0.0
├─ is-regex@1.0.5
├─ is-typedarray@1.0.0
├─ is-utf8@0.2.1
├─ is-windows@1.0.2
├─ isarray@1.0.0
├─ isexe@2.0.0
├─ isstream@0.1.2
├─ jade@1.11.0
├─ js-base64@2.5.2
├─ js-tokens@3.0.2
├─ js-yaml@3.14.0
├─ json-schema-traverse@0.4.1
├─ json-schema@0.2.3
├─ json-stringify-safe@5.0.1
├─ jsonparse@1.3.1
├─ JSONStream@1.3.5
├─ jsprim@1.4.1
├─ jstransformer@0.0.2
├─ lazy-cache@1.0.4
├─ load-json-file@1.1.0
├─ locate-path@3.0.0
├─ lodash.assignin@4.2.0
├─ lodash.bind@4.2.1
├─ lodash.defaults@4.2.0
├─ lodash.filter@4.6.0
├─ lodash.flatten@4.4.0
├─ lodash.foreach@4.5.0
├─ lodash.map@4.6.0
├─ lodash.merge@4.6.2
├─ lodash.pick@4.4.0
├─ lodash.reduce@4.6.0
├─ lodash.reject@4.6.0
├─ lodash.some@4.6.0
├─ lodash@4.17.15
├─ longest@1.0.1
├─ loose-envify@1.4.0
├─ loud-rejection@1.6.0
├─ lower-case@1.1.4
├─ map-obj@1.0.1
├─ map-visit@1.0.0
├─ markdown@0.5.0
├─ marked@0.3.19
├─ math-random@1.0.4
├─ meow@3.7.0
├─ micromatch@3.1.10
├─ mime-db@1.44.0
├─ mime-types@2.1.27
├─ mime@1.6.0
├─ minimatch@3.0.4
├─ minimist@1.2.5
├─ mixin-deep@1.3.2
├─ mkdirp@0.5.5
├─ moment-timezone@0.5.31
├─ moment@2.26.0
├─ morgan@1.10.0
├─ mv@2.1.1
├─ nan@2.14.1
├─ nanomatch@1.2.13
├─ ncp@2.0.0
├─ negotiator@0.6.2
├─ nib@1.1.2
├─ no-case@2.3.2
├─ node-fingerprint@0.0.2
├─ node-gyp@3.8.0
├─ node-sass@4.14.1
├─ nopt@3.0.6
├─ normalize-package-data@2.5.0
├─ normalize-path@2.1.1
├─ npmlog@4.1.2
├─ nth-check@1.0.2
├─ number-is-nan@1.0.1
├─ nunjucks@3.2.1
├─ oauth-sign@0.9.0
├─ object-copy@0.1.0
├─ object.omit@2.0.1
├─ opn@4.0.2
├─ os-tmpdir@1.0.2
├─ osenv@0.1.5
├─ p-limit@2.3.0
├─ p-locate@3.0.0
├─ p-try@2.2.0
├─ parse-glob@3.0.4
├─ parse-json@2.2.0
├─ pascalcase@0.1.1
├─ path-dirname@1.0.2
├─ path-exists@2.1.0
├─ path-parse@1.0.6
├─ path-type@1.1.0
├─ performance-now@2.1.0
├─ picomatch@2.2.2
├─ pinkie@2.0.4
├─ posix-character-classes@0.1.1
├─ preserve@0.2.0
├─ pretty-hrtime@1.0.3
├─ process-nextick-args@2.0.1
├─ promise@6.1.0
├─ pseudomap@1.0.2
├─ psl@1.8.0
├─ pug-attrs@2.0.4
├─ pug-code-gen@2.0.2
├─ pug-filters@3.1.1
├─ pug-lexer@4.1.0
├─ pug-linker@3.0.6
├─ pug-load@2.0.12
├─ pug-parser@5.0.1
├─ pug-strip-comments@1.0.4
├─ pug@2.0.4
├─ punycode@2.1.1
├─ qs@6.5.2
├─ randomatic@3.1.1
├─ range-parser@1.2.1
├─ read-pkg-up@1.0.1
├─ read-pkg@1.1.0
├─ readable-stream@2.3.7
├─ readdirp@2.2.1
├─ redent@1.0.0
├─ regenerator-runtime@0.11.1
├─ regex-cache@0.4.4
├─ remove-trailing-separator@1.1.0
├─ repeating@2.0.1
├─ request@2.88.2
├─ require-directory@2.1.1
├─ require-main-filename@2.0.0
├─ resolve-url@0.2.1
├─ ret@0.1.15
├─ right-align@0.1.3
├─ safe-json-stringify@1.2.0
├─ safer-buffer@2.1.2
├─ sass-graph@2.2.5
├─ sax@1.2.4
├─ scss-tokenizer@0.2.3
├─ semver@6.3.0
├─ send@0.17.1
├─ serve-static@1.14.1
├─ set-blocking@2.0.0
├─ set-value@2.0.1
├─ setprototypeof@1.1.1
├─ snapdragon-node@2.1.1
├─ snapdragon-util@3.0.1
├─ source-map-resolve@0.5.3
├─ source-map-url@0.4.0
├─ spdx-correct@3.1.1
├─ spdx-exceptions@2.3.0
├─ split-string@3.1.0
├─ sprintf-js@1.1.2
├─ sshpk@1.16.1
├─ static-extend@0.1.2
├─ stdout-stream@1.4.1
├─ string_decoder@1.1.1
├─ strip-bom@2.0.0
├─ striptags@2.2.1
├─ stylus@0.54.7
├─ supports-color@2.0.0
├─ swig-extras@0.0.1
├─ swig-templates@2.0.3
├─ swig@1.4.2
├─ tar@2.2.2
├─ text-table@0.2.0
├─ through@2.3.8
├─ titlecase@1.1.3
├─ to-fast-properties@1.0.3
├─ to-object-path@0.3.0
├─ to-regex-range@2.1.1
├─ toidentifier@1.0.0
├─ token-stream@0.0.1
├─ tough-cookie@2.5.0
├─ transformers@2.1.0
├─ trim-newlines@1.0.0
├─ true-case-path@1.0.3
├─ tunnel-agent@0.6.0
├─ tweetnacl@0.14.5
├─ uglify-js@2.8.29
├─ uglify-to-browserify@1.0.2
├─ union-value@1.0.1
├─ unpipe@1.0.0
├─ unset-value@1.0.0
├─ upath@1.2.0
├─ upper-case@1.1.3
├─ uri-js@4.2.2
├─ use@3.1.1
├─ util-deprecate@1.0.2
├─ uuid@3.4.0
├─ validate-npm-package-license@3.0.4
├─ vary@1.1.2
├─ verror@1.10.0
├─ void-elements@2.0.1
├─ warehouse@2.2.0
├─ which-module@2.0.0
├─ which@1.3.1
├─ wide-align@1.1.3
├─ with@4.0.3
├─ wrap-ansi@5.1.0
├─ y18n@4.0.0
├─ yallist@2.1.2
└─ yargs-parser@13.1.2
Done in 181.40s.
```

## 然后，修改themes/melody/_config.yml

```yml
local_search:
  enable: false
```

为

```yml
local_search:
  enable: true # or false
  labels:
    input_placeholder: Search for Posts
    hits_empty: "We didn't find any results for the search: ${query}" # if there are no result
```

完成

## 其他

执行sudo apt-get update的时候报一些URI有问题。这些网址在下面这个目录中
/etc/apt/sources.list.d/

如果把.list文件改扩展名，可以修复错误，但是会提示以下信息。有洁癖的可以删掉不要的文件。

N: 忽略‘d-apt.list.bak1’(于目录‘/etc/apt/sources.list.d/’)，鉴于它的文件扩展名无效
N: 忽略‘fcitx-team-ubuntu-nightly-bionic.list.bak1’(于目录‘/etc/apt/sources.list.d/’)，鉴于它的文件扩展名无效
