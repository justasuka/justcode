---
title: format emoji in markdown
date: 2021-12-03 15:53:57
tags: ['markdown']
---

# 转换markdown中的emoji
>  replace :emoji: to real UTF-8 emojis in text.

​	在Typora中写的emoji 可以方便的写成`:smile:`-->:smiley: 这样来写。但是在不能解析的地方，显示的不是emoji :smiley: 而是plain text `:smiley:`。

### 需求:satellite:
转换markdown中的emoji

### 可用的工具:tada:

[remark](https://github.com/remarkjs/remark)

> remark is a popular tool that transforms markdown with plugins. These plugins can inspect and change your markup. You can use remark on the server, the client, CLIs, deno, etc.

配套的插件[remark-emoji](https://github.com/rhysd/remark-emoji)

> Remark markdown transformer to replace :emoji: in text

或者

[markdown-it](https://github.com/markdown-it/markdown-it)

> Markdown parser, done right. 100% CommonMark support, extensions, syntax plugins & high speed

加上插件[markdown-it-emoji](https://github.com/markdown-it/markdown-it-emoji)

> Emoji syntax plugin for markdown-it markdown parser

### 解决办法:door:

使用`remark-cli`来格式化

```shell
npm install remark-cli remark-emoji
```

在package.json中的配置

```json
  "scripts": {
    "format": "remark . --output"
  },
    "remarkConfig": {
    "settings": {
      "bullet": "*"
    },
    "plugins": [
      "remark-emoji"
    ]
  }
  
```

remark中有其他许多可配置的格式化项。

配置好后直接`npm run format`格式化当前目录下及子目录下的`.md`文件。

### 限制

部分格式在一些本身有特定格式的md里会冲突(例如hexo里面，但是这些工具的格式化配置又没玩明白。所以采取他法)。在hexo中本身有许多[插件](https://hexo.io/plugins/)

[markdown-it-plus](https://github.com/CHENXCHEN/hexo-renderer-markdown-it-plus)

```sh
npm un hexo-renderer-marked --save
npm i hexo-renderer-markdown-it-plus --save
```

_config.yml中的配置

```yml
markdown_it_plus:
    highlight: true
    html: true
    xhtmlOut: true
    breaks: true
    langPrefix:
    linkify: true
    typographer:
    quotes: “”‘’
    plugins:
        - plugin:
            name: markdown-it-emoji
            enable: true
```

