---
layout: fragment
title: 编译jekyll报错
tags: jekyll
---

今天编译 `2023-6-12-github-sphinx-read-the-docs-getting-start-1` 文档的时候，github 提示了报错 Error: Liquid syntax error (line 99): Unknown tag 'extends'，查阅发现该文档有如下代码段：

``` html
\```
{%extends "!layout.html" %}
{% block extrahead %}
    <link href="{{ pathto("_static/style.css", True) }}" rel="stylesheet" type="text/css">
{% endblock %}
\```
```

查阅资料[ruby - Getting an "Liquid Exception: Liquid syntax error" while using Jekyll - Stack Overflow](https://stackoverflow.com/questions/52324134/getting-an-liquid-exception-liquid-syntax-error-while-using-jekyll)得知，Liquid 会尝试处理您的源代码，尤其是 jinja2 控制标记，为此您需要告诉 Liquid 避免使用原始标记进行处理。
