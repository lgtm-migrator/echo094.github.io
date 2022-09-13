---
layout: post
title:  "Javascript的AST处理"
date:   2022-09-12 00:00:00 +0000
tags: ast
---

通过AST，我们可以对代码进行格式化，也可以用来简化混淆后的JS代码。

## 1 背景知识

在线混淆网站：

* [obfuscator](https://obfuscator.io/)
* [SO JSON](https://www.sojson.com/)
* [JS加密](https://www.jsjiami.com/)

在线AST分析：

* [AST Explorer](https://astexplorer.net/)

资料：

* [NightTeam/JavaScriptAST](https://github.com/NightTeam/JavaScriptAST)
* [【JS 逆向百例】AST 脱混淆实战，某 ICP 备案号查询接口 jsjiami v6 分析](https://zhuanlan.zhihu.com/p/520162203)

这里使用[Babel](https://babeljs.io/)对代码进行处理，需要用到的工具如下：

* `@babel/parser` 根据代码建立AST
* `@babel/generator` 从AST生成代码
* `@babel/traverse` 遍历并修改AST
* `@babel/types` AST中的节点类型

详细的使用介绍可以查看手册：
[Babel Plugin Handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md)

各种代码类型的定义以及参数说明见：
[@babel/types](https://babeljs.io/docs/en/babel-types)

## 2 应用

这里拿一个常见的混淆方式练习，这种加密方式目前有一个纯正则的解密方案：
[NXY666/JsjiamiV6-Decryptor](https://github.com/NXY666/JsjiamiV6-Decryptor)，
现在尝试将其转为AST版本。

1. 净化代码
   
   这个步骤是去除注释， 在AST中会自动分离注释。

2. 解除全局加密
   
   首先将`![]`和`!![]`分别转换为`true`和`false`，为后续的分割做准备，在AST中不需要。
   
   然后将各语句分为：签名信息，预处理函数，解密函数，验证函数，常规语句。

   如果混淆后的文件没有被二次修改，那么前3个非空语句分别为：签名信息，预处理函数，解密函数。
   * 签名信息类型为`VariableDeclaration`；
   * 预处理函数的不定义变量或函数，直接对签名信息的变量进行修改；
   * 解密函数的类型为`VariableDeclaration`或`FunctionDeclaration`。
   
   我们只需要将这3个语句丢到同一个VM环境中，再遍历节点将所有调用解密函数的节点替换为值。

3. 解除代码块加密（递归进行下述操作）
   
   首先删除禁止格式化的代码，一般在内容代码前方。
   这里并没有找到样本，先跳过。
   
   然后还原代码块内统一收集的字符串、函数调用（二元函数，比较函数，函数调用）。
   * 首先对每个Object进行判断，如果所有的成员都符合上述特征，则说明可能为代码块加密。
   * 然后遍历作用域，替换上述元素，并统计这些元素是否都使用到。

4. 清理死代码
   
   [TBC]

5. 解除环境限制
   
   [TBC]

6. 提高代码可读性
   
   [TBC]

7. 格式化代码
   
   [TBC]


