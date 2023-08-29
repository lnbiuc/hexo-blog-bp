---
title: markdown-it解析HTML及目录
date: 2022/9/28 23:56:46
updated: 2023/3/23 22:57:27
categories:
- FrontEnd
tags:
- Vue.js
---

# 效果图
![image-20220928235607296](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220928235607296.png)
# 安装所有依赖

```sh
npm i markdown-it highlight.js markdown-it-emoji markdown-it-ins markdown-it-footnote markdown-it-sub markdown-it-sup markdown-it-deflist markdown-it-abbr markdown-it-mark markdown-it-anchor markdown-it-multimd-table uslug markdown-it-smartarrows tocbot clipboard
```

# markdown-it配置

## 解析方法

> 接收md字符串，返回生成的Html代码

```javascript
import MarkdownIt from 'markdown-it';
import hljs from 'highlight.js';
import emoji from 'markdown-it-emoji';
import ins from 'markdown-it-ins';
import footnote from 'markdown-it-footnote';
import sub from 'markdown-it-sub';
import sup from 'markdown-it-sup';
import list from 'markdown-it-deflist';
import addr from 'markdown-it-abbr';
import mark from 'markdown-it-mark';
import anchor from 'markdown-it-anchor';
import multimd from 'markdown-it-multimd-table';
import uslug from 'uslug';
import smartarrows from 'markdown-it-smartarrows';

export function mdStrToHTML(htmlStr) {
    const md = new MarkdownIt({
        html: true,
        linkify: true,
        typographer: true,
        highlight: function (str, lang) {
            // 当前时间加随机数生成唯一的id标识
            const codeIndex = parseInt(Date.now()) + Math.floor(Math.random() * 10000000);
            // 复制功能主要使用的是 clipboard.js
            let html = `<a class="copy-btn" type="button" data-clipboard-action="copy" data-clipboard-target="#copy${codeIndex}"><i class="copy icon"></i></a>`;
            const linesLength = str.split(/\n/).length - 1;
            // 生成行号
            let linesNum = '<span aria-hidden="true" class="line-numbers-rows">';
            for (let index = 0; index < linesLength; index++) {
                linesNum = linesNum + '<span></span>';
            }
            linesNum += '</span>';
            if (lang && hljs.getLanguage(lang)) {
                try {
                    const preCode = hljs.highlight(lang, str, true).value;
                    html = html + preCode;
                    if (linesLength) {
                        html += '<b class="name">' + lang + '</b>';
                    }
                    return `<pre class="hljs"><code>${html}</code>${linesNum}</pre><textarea style="position: absolute;top: -9999px;left: -9999px;z-index: -9999;" id="copy${codeIndex}">${str.replace(
                        /<\/textarea>/g,
                        '&lt;/textarea>'
                    )}</textarea>`;
                } catch (error) {
                    console.log(error);
                }
            }

            const preCode = md.utils.escapeHtml(str);
            html = html + preCode;
            return `<pre class="hljs"><code>${html}</code>${linesNum}</pre><textarea style="position: absolute;top: -9999px;left: -9999px;z-index: -9999;" id="copy${codeIndex}">${str.replace(
                /<\/textarea>/g,
                '&lt;/textarea>'
            )}</textarea>`;
        },
    });
    md.use(emoji);
    md.use(ins);
    md.use(footnote);
    md.use(sub);
    md.use(sup);
    md.use(list);
    md.use(addr);
    md.use(mark);
    md.use(multimd, {
        multiline: false,
        rowspan: false,
        headerless: false,
        multibody: true,
        aotolabel: true,
    });
    md.use(smartarrows);

    const uslugify = s => uslug(s);

    md.use(anchor, {
        permalink: false,
        permalinkSpace: false,
        slugify: uslugify,
    });
    md.renderer.rules.image = function (tokens, idx, options, env, slf) {
        const token = tokens[idx];
        token.attrs[token.attrIndex('alt')][1] = slf.renderInlineAsText(
            token.children,
            options,
            env
        );
        token.attrSet('style', 'max-width:100%');
        return slf.renderToken(tokens, idx, options);
    };

    return md.render(htmlStr);
}
```

## 添加到页面

> 将渲染好的Html代码添加到页面中，使用vue的v-html标签

```html
<div class="content" v-html="content"></div>
```

## 方法调用

> 在页面`created()`被创建的时候，从路径获取文章的id，发送请求，获取相应结果，调用方法，渲染到页面。

```javascript
created() {
	this.articleId = this.$route.params.articleId
	getArticleByArticleId(this.articleId).then((res) => {
		this.blog = res.data.data
		this.content = mdStrToHTML(this.blog.content)
	})
},
```

## 复制代码按钮

>  使用`clipboard`来复制，复制成功弹出`elementUI`的提示消息

```javascript
mounted() {
	this.clipboard = new Clipboard('.copy-btn')
	// 复制成功失败的提示
	this.clipboard.on('success', (e) => {
		ElMessage.success('复制成功')
	})
	this.clipboard.on('error', (e) => {
		ElMessage.error('复制失败')
	})
}
```

> 在离开页面时需要销毁复制和加载目录的方法，如果没有及时销毁会多次复制，目录会一直监听滚动事件

```javascript
beforeDestroy() {
	this.clipboard.destroy()
	tocbot.destroy()
},
destroy() {
	this.clipboard.destroy()
	tocbot.destroy()
},
beforeRouteLeave() {
	this.clipboard.destroy()
	tocbot.destroy()
}
```

## 样式

> 代码块样式、复制按钮样式、代码块行号样式

```css
import 'highlight.js/styles/github-dark.css';
<style lang="scss">
pre.hljs:hover {
    b.name {
        display: none;
        transition: all 500ms;
    }

    .copy-btn {
        display: block;
        transition: all 500ms;
    }
}

pre.hljs {
    padding: 12px 2px 12px 40px !important;
    border-radius: 5px !important;
    position: relative;
    font-size: 14px !important;
    line-height: 22px !important;
    overflow: hidden !important;
    background-color: #161b22 !important;

    code {
        display: block !important;
        margin: 0 10px !important;
        overflow-x: auto !important;

        &::-webkit-scrollbar {
            z-index: 11;
            width: 6px;
        }

        &::-webkit-scrollbar:horizontal {
            height: 6px;
        }

        &::-webkit-scrollbar-thumb {
            border-radius: 7px;
            width: 6px;
            background: #666;
        }

        &::-webkit-scrollbar-corner,
        &::-webkit-scrollbar-track {
            background: #1e1e1e;
        }

        &::-webkit-scrollbar-track-piece {
            background: #1e1e1e;
            width: 6px;
        }
    }

    .line-numbers-rows {
        position: absolute;
        pointer-events: none;
        top: 12px;
        bottom: 12px;
        left: 0;
        font-size: 100%;
        width: 40px;
        text-align: center;
        letter-spacing: -1px;
        border-right: 1px solid rgba(0, 0, 0, 0.66);
        user-select: none;
        counter-reset: linenumber;

        span {
            pointer-events: none;
            display: block;
            counter-increment: linenumber;

            &:before {
                content: counter(linenumber);
                color: #999;
                display: block;
                text-align: center;
            }
        }
    }

    b.name {
        position: absolute;
        top: 2px;
        right: 8px;
        z-index: 10;
        color: #999;
        pointer-events: none;
    }

    .copy-btn {
        position: absolute;
        top: 0;
        right: 0;
        z-index: 10;
        border-radius: 5px;
        border: 1px solid #5b5b5b;
        padding: 5px 5px;
        color: #5b5b5b;
        font-size: 20px;
        margin: 5px 5px;
        display: none;
    }

    .copy-btn:hover {
        color: #fff;
        border: 1px solid #fff;
        transition: all 200ms;
    }

    .copy-btn:active {
        color: green;
        border: 1px solid green;
    }

    .copy-btn:focus {
        color: green;
        border: 1px solid green;
    }
}
```

# 目录配置

> 使用tocbot提取h标签生成目录，在文章加载完毕之后调用该方法

```javascript
export function renderToc() {
	// 生成目录插件初始化
	tocbot.init({
		// Where to render the table of contents.
		tocSelector: '.js-toc', // 生成的目录放在哪儿 , 这里用css选中
		// Where to grab the headings to build the table of contents.
		contentSelector: '.contnent', // 生成目录的源在那儿 , 就是给谁生成目录
		// Which headings to grab inside of the contentSelector element.
		headingSelector: 'h1, h2, h3', // 能给生成的几级标题
		// For headings inside relative or absolute positioned containers within content.
		hasInnerContainers: true,
		// Smooth scrolling enabled.
		scrollSmooth: true,
		// Smooth scroll duration.
		scrollSmoothDuration: 420,
		// 到顶部导航条的距离
		scrollSmoothOffset: -70,
		// Headings offset between the headings and the top of the document (this is meant for minor adjustments).
		// Can also be used to account for scroll height discrepancies from the use of css scroll-padding-top
		headingsOffset: 70,
	});
}
```

## 方法调用

> 可能会在文章未完全加载时就调用生成目录，导致需要的dom没有渲染完成，会报错所以这里通过不断调用知道生成成功。

```javascript
checkDom() {
	var timer = null
	if (
		window.document.querySelector('.js-toc') &&
		window.document.querySelector('.contnent')
	) {
	this.initTocbot()
	if (!timer) {
		clearTimeout(timer)
			}
	} else {
	//  自我调用
	timer = setTimeout(this.checkDom, 0)
	}
}
```

> 在页面加载完毕之后在调用

```javascript
mounted() {
    this.checkDom()
}
```


