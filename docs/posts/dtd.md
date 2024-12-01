---
date: 2024-06-07
authors:
  - leoalasiga
tags:
 - dtd
 - xml
comments: true
---

# 怎么编写DTD文件

编写DTD（文档类型定义，Document Type Definition）是一种方式来定义XML文档的结构、元素、属性以及元素间的关系。DTD定义可以内嵌在XML文档中，也可以作为一个外部文件供多个XML文档引用。下面是编写DTD的基本步骤和语法示例：

### 基本结构

DTD可以分为两种形式：内部DTD和外部DTD。

#### 内部DTD

直接在XML文档中定义DTD，位于XML声明之后，文档元素之前。

```Xml
<?xml version="1.0"?>
<!DOCTYPE 根元素名称 [
    <!-- DTD的内容 -->
]>
<根元素名称>
    <!-- XML文档内容 -->
</根元素名称>
```

#### 外部DTD

外部DTD存储在单独的.dtd文件中，XML文档通过DOCTYPE声明引用它。

```Xml
<?xml version="1.0"?>
<!DOCTYPE 根元素名称 SYSTEM "外部DTD文件路径.dtd">
<根元素名称>
    <!-- XML文档内容 -->
</根元素名称>
```

或者使用PUBLIC关键字指定DTD的公共标识符和系统标识符：

```Xml
<!DOCTYPE 根元素名称 PUBLIC "DTD公共标识符" "外部DTD文件路径.dtd">
```

### DTD元素定义

- **元素声明**：
  - **无子元素**：`<!ELEMENT 元素名称 EMPTY>`
  - **有文本内容**：`<!ELEMENT 元素名称 (#PCDATA)>`
  - **有子元素**：`<!ELEMENT 元素名称 (子元素1, 子元素2, ..., 子元素N)>`
  - **子元素可选**：在子元素后加问号`?`，表示该元素可有可无，如`(子元素1, 子元素2?)`
  - **子元素可重复**：在子元素后加星号`*`或加加号`+`，表示该元素可以重复多次或至少一次，如`(子元素1, 子元素2*)`
- **属性声明**：
  - 使用`<!ATTLIST 元素名称 属性名 属性类型 默认值>`格式。
  - 属性类型可以是`CDATA`（任意文本）、`ID`（唯一标识符）、`IDREF`（引用其他元素的ID）、`ENUMERATED`（枚举值）等。
  - 默认值可以是`#REQUIRED`（必须提供）、`#IMPLIED`（可选）、具体值等。

### 示例

假设我们定义一个简单的图书目录DTD，包含书籍（book）和章节（chapter）。

**外部DTD文件（books.dtd）**:

```
<!ELEMENT catalog (book+)>
<!ELEMENT book (title, author+, chapter*)>
<!ELEMENT title (#PCDATA)>
<!ELEMENT author (#PCDATA)>
<!ELEMENT chapter (#PCDATA)>
```

**XML文档**:

```
<?xml version="1.0"?>
<!DOCTYPE catalog SYSTEM "books.dtd">
<catalog>
    <book>
        <title>示例书名</title>
        <author>作者A</author>
        <author>作者B</author>
        <chapter>第一章</chapter>
        <chapter>第二章</chapter>
    </book>
    <!-- 更多书籍... -->
</catalog>
```

这个例子展示了如何定义一个XML文档的结构，包括哪些元素是必需的，哪些是可选的，以及它们之间的关系。通过DTD，可以确保XML文档遵循特定的结构和规则。
