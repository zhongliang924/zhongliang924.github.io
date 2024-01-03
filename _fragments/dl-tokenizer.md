---
layout: fragment
title: 语言模型中的Tokenizer
tags: DL
---

在语言模型中，Tokenizer是NLP大模型最基础的组件，其作用是将文本分割成独立的token列表，比如单词或者子词，进而转换成输入的向量以便计算机能更好理解和处理文本数据。Tokenizer是自然语言处理流程中的一部分，通常用于将原始文本转换成模型可以理解的输入格式。以下是一些常见的Tokenizer类型：

- 基于字符级的Tokenizer：将文本分割成单个字符，不考虑单词的语义
- 基于单词的Tokenizer：将文本分割成单词作为标记，考虑单词之间的语义关系
- Subword Tokenizer：介于字符级和单词之间的 Tokenizer，它将单词分割成更小的部分，通常是子词，有助于处理不同形式的单词，尤其对于词根、后缀等形态变化较多的语言。
- BERT Tokenizer：针对 Transformer 模型设计的 Tokenizer，不仅考虑了单词的边界，还将单词分割成子词片段，有助于模型能更好理解复杂的语言结构和语义关系。