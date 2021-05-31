---
layout: post
title: "MarkDown To PDF"
subtitle: '最方便的Markdown转PDF流程'
author: "Song"
header-style: text
tags:
  - markdown
  - pdf
  - pandoc
  - mdout
---

## 简介
Markdown 转 PDF 有很多方案，但是操作最方便且支持配置最多还是 Pandoc。Pandoc 不能直接转 PDF，需要通过LaTeX转为 PDF，LaTeX引擎的样式配置比较麻烦，且对中文的支持不好，所以最终选用 Markdown 转 HTML，再 HTML 转 PDF。HTML 转 PDF 有很多方案，比如 wkhtmltopdf、markdown-pdf、prince、weasyprint、mdout 最终选用 mdout 操作简便，且自定义样式方便。

> https://github.com/alanshaw/markdown-pdf

## Pandoc
Pandoc 可以将文档在 Markdown、LaTeX、reStructuredText、HTML、Word docx 等多种标记格式之间相互转换，并支持输出 PDF、EPUB、HTML 幻灯片等多种格式。该程序被称为格式转换界的 “瑞士军刀”。

仓库：https://github.com/jgm/pandoc

Pandoc 转 html 制定模板

```shell
--css templates.css
```

测试发现 Typora 开源的 Github 主题效果最好，https://github.com/typora/typora-default-themes，但这个版本很久未更新，还需要优化。

```css
html {
    font-size: 16px;
	max-width: 210mm;
    padding: 12mm 16mm 12mm 16mm;
    margin: 0 auto;
}

tbody, table {
	border-left: 1px solid #cccccc;
	border-top: 1px solid #cccccc;
}

table tr {
    border: 1px solid #cccccc;
    margin: 0;
    padding: 0;
}

table tr th {
    font-weight: bold;
    border-right: 1px solid #cccccc;
    border-bottom: 1px solid #cccccc;
    text-align: left;
    margin: 0;
    padding: 6px 13px;
}

table tr td {
	border-right: 1px solid #cccccc;
    border-bottom: 1px solid #cccccc;
    text-align: left;
    margin: 0;
    padding: 6px 13px;
}

code, div{
	// border: 1px solid #ddd;
    background-color: #f8f8f8;
    border-radius: 3px;
    padding: 0;
    font-family: Consolas, "Liberation Mono", Courier, monospace;
    padding: 2px 4px 2px 4px;
    font-size: 0.9em;
}
```



> 参考：https://www.jianshu.com/p/6ba04f669d0b 
>
> https://www.jianshu.com/p/7f9a9ff053bb
>
> https://jdhao.github.io/2017/12/10/pandoc-markdown-with-chinese/
>
> https://github.com/kjhealy/pandoc-templates/
>
> 官方模板：https://github.com/jgm/pandoc/wiki/User-contributed-templates

## wkhtmltopdf

> https://wkhtmltopdf.org/index.html
>
> https://www.jianshu.com/p/9be654799449

## mdout

```shell
mdout test.html -t pdf
```

> https://github.com/JabinGP/mdout

## 模板

https://github.com/jgm/pandoc/wiki/User-contributed-templates

## 总结

> https://github.com/skyArony/md-style