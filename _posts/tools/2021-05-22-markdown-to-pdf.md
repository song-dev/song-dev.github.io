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
Markdown 转 PDF 有很多方案，但是操作最方便且支持配置最多还是 Pandoc。Pandoc 不能直接转 PDF，需要通过LaTeX转为 PDF，LaTeX引擎的样式配置比较麻烦，且对中文的支持不好，所以最终选用 Markdown 转 HTML，再 HTML 转 PDF。HTML 转 PDF 有很多方案，比如 wkhtmltopdf、mdout 最终选用 mdout 操作简便，且自定义样式方便。

## Pandoc
Pandoc 可以将文档在 Markdown、LaTeX、reStructuredText、HTML、Word docx 等多种标记格式之间相互转换，并支持输出 PDF、EPUB、HTML 幻灯片等多种格式。该程序被称为格式转换界的 “瑞士军刀”。

> https://www.jianshu.com/p/6ba04f669d0b

## wkhtmltopdf

> https://wkhtmltopdf.org/index.html

## mdout

> https://github.com/JabinGP/mdout

## 总结

> https://github.com/skyArony/md-style