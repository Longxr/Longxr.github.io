---
title: QtCreator代码格式化
categories:
  - Qt
tags:
  - clang format
date: 2020-01-07 15:58:51
---

<Excerpt in index | 摘要> 
QtCreator支持多种格式化工具， 我使用的是clang-format，这个工具能够自动化格式C/C++/Obj-C代码，支持多种代码风格（Google, Chromium, LLVM, Mozilla, WebKit），同时也支持自定义风格 <!-- more -->
<The rest of contents | 余下全文>

## 前置条件
如果使用其他插件想要看怎么设置的话可以看Qt的[官方文档](https://doc.qt.io/qtcreator/creator-beautifier.html)，我这里就说下clang-format需要做的步骤。
1. [下载llvm并安装](http://releases.llvm.org/download.html)，如果除了格式化以外不用的话，安装的时候最好不要加到环境变量
2. 在Qt Creator的关于插件中启用Beautifier插件，启用插件需要重启下Creator。
![启用Beautifier插件](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/QtCreator-Code-Format_01.png)

## 使用配置
关于插件的配置在工具->选项->Beautifier中设置， 设置好clang-format的路径 `C:\Program Files\LLVM\bin\clang-format.exe`。

### 通过.clang-format文件使用
1. 在插件的配置中*Options*选择*Use predefined style*，下拉选择File。
![通过文件使用](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/QtCreator-Code-Format_02.png)
2. 写一个.clang-format文件放到项目中和最外层的.pro同一级目录，下面是带中文注释的.code-format，也可以去[clang-format官方说明](http://clang.llvm.org/docs/ClangFormatStyleOptions.html)中查看。

```
# 语言: None, Cpp, Java, JavaScript, ObjC, Proto, TableGen, TextProto
Language: Cpp
# 未在配置中专门设置的所有选项所使用的样式
BasedOnStyle:  Google
# 访问说明符(public、private等)的偏移
AccessModifierOffset: -4
# 开括号(开圆括号、开尖括号、开方括号)后的对齐: Align, DontAlign, AlwaysBreak(总是在开括号后换行)
AlignAfterOpenBracket: AlwaysBreak
# 连续赋值时，对齐所有等号
AlignConsecutiveAssignments: true
# 连续声明时，对齐所有声明的变量名
AlignConsecutiveDeclarations: true
# 左对齐逃脱换行(使用反斜杠换行)的反斜杠
AlignEscapedNewlines: Right
# 水平对齐二元和三元表达式的操作数
AlignOperands: true
# 对齐连续的尾随的注释
AlignTrailingComments: true
# 允许函数声明的所有参数在放在下一行
AllowAllParametersOfDeclarationOnNextLine: true
# 允许短的块放在同一行
AllowShortBlocksOnASingleLine: false
# 允许短的case标签放在同一行
AllowShortCaseLabelsOnASingleLine: false
# 允许短的函数放在同一行: None, InlineOnly(定义在类中), Empty(空函数), Inline(定义在类中，空函数), All
AllowShortFunctionsOnASingleLine: Empty
# 允许短的lambda放在同一行
AllowShortLambdasOnASingleLine: Empty
# 允许短的if语句保持在同一行
AllowShortIfStatementsOnASingleLine: true
# 允许短的循环保持在同一行
AllowShortLoopsOnASingleLine: false
# 总是在定义返回类型后换行(deprecated)
AlwaysBreakAfterDefinitionReturnType: None
# 总是在返回类型后换行: None, All, TopLevel(顶级函数，不包括在类中的函数),
#   AllDefinitions(所有的定义，不包括声明), TopLevelDefinitions(所有的顶级函数的定义)
AlwaysBreakAfterReturnType: None
# 总是在多行string字面量前换行
AlwaysBreakBeforeMultilineStrings: false
# 总是在template声明后换行
AlwaysBreakTemplateDeclarations: true
# false表示函数实参要么都在同一行，要么都各自一行
BinPackArguments: true
# false表示所有形参要么都在同一行，要么都各自一行
BinPackParameters: true
# 在二元运算符前换行: None(在操作符后换行), NonAssignment(在非赋值的操作符前换行), All(在操作符前换行)
BreakBeforeBinaryOperators: All
# 在大括号前换行: Attach(始终将大括号附加到周围的上下文), Linux(除函数、命名空间和类定义，与Attach类似),
#   Mozilla(除枚举、函数、记录定义，与Attach类似), Stroustrup(除函数定义、catch、else，与Attach类似),
#   Allman(总是在大括号前换行), GNU(总是在大括号前换行，并对于控制语句的大括号增加额外的缩进), WebKit(在函数前换行), Custom
#   注：这里认为语句块也属于函数
BreakBeforeBraces: Mozilla
# 继承列表的逗号前换行
BreakBeforeInheritanceComma: false
# 在三元运算符前换行
BreakBeforeTernaryOperators: true
# 在构造函数的初始化列表的逗号前换行
BreakConstructorInitializersBeforeComma: false
# 初始化列表前换行
BreakConstructorInitializers: BeforeComma
# Java注解后换行
BreakAfterJavaFieldAnnotations: false

BreakStringLiterals: true
# 每行字符的限制，0表示没有限制
ColumnLimit:     160
# 描述具有特殊意义的注释的正则表达式，它不应该被分割为多行或以其它方式改变
CommentPragmas:  '^ IWYU pragma:'
# 紧凑 命名空间
CompactNamespaces: false
# 构造函数的初始化列表要么都在同一行，要么都各自一行
ConstructorInitializerAllOnOneLineOrOnePerLine: true
# 构造函数的初始化列表的缩进宽度
ConstructorInitializerIndentWidth: 4
# 延续的行的缩进宽度
ContinuationIndentWidth: 4
# 去除C++11的列表初始化的大括号{后和}前的空格
Cpp11BracedListStyle: false
# 继承最常用的指针和引用的对齐方式
DerivePointerAlignment: false
# 关闭格式化
DisableFormat:   false
# 自动检测函数的调用和定义是否被格式为每行一个参数(Experimental)
ExperimentalAutoDetectBinPacking: false
# 固定命名空间注释
FixNamespaceComments: true
# 需要被解读为foreach循环而不是函数调用的宏
ForEachMacros:   
  - foreach
  - Q_FOREACH
  - BOOST_FOREACH

IncludeBlocks:   Preserve
# 对#include进行排序，匹配了某正则表达式的#include拥有对应的优先级，匹配不到的则默认优先级为INT_MAX(优先级越小排序越靠前)，
#   可以定义负数优先级从而保证某些#include永远在最前面
IncludeCategories: 
  - Regex:           '^"(llvm|llvm-c|clang|clang-c)/'
    Priority:        2
  - Regex:           '^(<|"(gtest|gmock|isl|json)/)'
    Priority:        3
  - Regex:           '.*'
    Priority:        1
IncludeIsMainRegex: '(Test)?$'
# 缩进case标签
IndentCaseLabels: true

IndentPPDirectives: None
# 缩进宽度
IndentWidth:     4
# 函数返回类型换行时，缩进函数声明或函数定义的函数名
IndentWrappedFunctionNames: false

JavaScriptQuotes: Leave

JavaScriptWrapImports: true
# 保留在块开始处的空行
KeepEmptyLinesAtTheStartOfBlocks: true
# 开始一个块的宏的正则表达式
MacroBlockBegin: ''
# 结束一个块的宏的正则表达式
MacroBlockEnd:   ''
# 连续空行的最大数量
MaxEmptyLinesToKeep: 1

# 命名空间的缩进: None, Inner(缩进嵌套的命名空间中的内容), All
NamespaceIndentation: All
# 使用ObjC块时缩进宽度
ObjCBlockIndentWidth: 4
# 在ObjC的@property后添加一个空格
ObjCSpaceAfterProperty: true
# 在ObjC的protocol列表前添加一个空格
ObjCSpaceBeforeProtocolList: true

PenaltyBreakAssignment: 2

PenaltyBreakBeforeFirstCallParameter: 19
# 在一个注释中引入换行的penalty
PenaltyBreakComment: 300
# 第一次在<<前换行的penalty
PenaltyBreakFirstLessLess: 120
# 在一个字符串字面量中引入换行的penalty
PenaltyBreakString: 1000
# 对于每个在行字符数限制之外的字符的penalty
PenaltyExcessCharacter: 1000000
# 将函数的返回类型放到它自己的行的penalty
PenaltyReturnTypeOnItsOwnLine: 60
# 指针和引用的对齐: Left, Right, Middle
PointerAlignment: Left

#RawStringFormats: 
#  - Delimiter:       pb
#    Language:        TextProto
#    BasedOnStyle:    google
# 允许重新排版注释
ReflowComments:  true
# 允许排序#include
SortIncludes:    true

SortUsingDeclarations: true
# 在C风格类型转换后添加空格
SpaceAfterCStyleCast: false
# 模板关键字后面添加空格
SpaceAfterTemplateKeyword: true
# 在赋值运算符之前添加空格
SpaceBeforeAssignmentOperators: true
# 开圆括号之前添加一个空格: Never, ControlStatements, Always
SpaceBeforeParens: ControlStatements
# 在空的圆括号中添加空格
SpaceInEmptyParentheses: false
# 在尾随的评论前添加的空格数(只适用于//)
SpacesBeforeTrailingComments: 4
# 在尖括号的<后和>前添加空格
SpacesInAngles:  false
# 在容器(ObjC和JavaScript的数组和字典等)字面量中添加空格
SpacesInContainerLiterals: true
# 在C风格类型转换的括号中添加空格
SpacesInCStyleCastParentheses: false
# 在圆括号的(后和)前添加空格
SpacesInParentheses: false
# 在方括号的[后和]前添加空格，lamda表达式和未指明大小的数组的声明不受影响
SpacesInSquareBrackets: false
# 标准: Cpp03, Cpp11, Auto
Standard:        Cpp11
# tab宽度
TabWidth:        4
# 使用tab字符: Never, ForIndentation, ForContinuationAndIndentation, Always
UseTab:          Never
```
### 通过在插件中添加代码使用
使用上面配置文件的话每次都需要拷贝一个文件到对应的目录下，工程多了或者临时想格式化一部分代码不方便，可以直接将配置写到插件的配置中。在插件的配置中*Options*选择*Use customized style*，新建一个配置文件写入配置。
![在配置中直接写](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/QtCreator-Code-Format_03.png)

由于不能在配置中写中文，并且配置文件中的*BasedOnStyle*可以设置默认样式，只需要写上和默认配置不同的部分，下面是相关配置:
```
# Defines the QtCreator style for automatic reformatting.
# http://clang.llvm.org/docs/ClangFormatStyleOptions.html

Language: Cpp
BasedOnStyle:  Google
Standard: Cpp11
AccessModifierOffset: -4
AlignConsecutiveAssignments: true
AlignConsecutiveDeclarations: true
AlwaysBreakTemplateDeclarations: true
BinPackArguments: true
BinPackParameters: true
BreakBeforeBinaryOperators: All
BreakBeforeBraces: Mozilla
ColumnLimit:     160
ContinuationIndentWidth: 4
ForEachMacros:   
  - foreach
  - Q_FOREACH
  - BOOST_FOREACH
IndentCaseLabels: true
IndentPPDirectives: None
IndentWidth:     4
```
## 快捷键设置
可以给格式化插件设置快捷键，我是给`Format at Cursor`设置了Alt+Shift+F (和VS Code的格式话相同)，使用的时候需要选中需要格式化的部分。
![快捷键设置](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/QtCreator-Code-Format_04.png)

至于为啥没有设置`Format Current File`设置，是因为格式化的时候会把带lambda表达式的connect函数整成下图的样子，写成这个样子可能会被打_(:з)∠)_，所以我选择格式化选中的部分。
![格式化之前](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/QtCreator-Code-Format_05.png)

![格式化之后](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/QtCreator-Code-Format_06.png)

*如果有什么办法可以改掉connect这种函数里带个lambda的格式的话记得评论和我说下呀*

## 相关链接
* [Clang-Format格式化选项介绍](https://blog.csdn.net/softimite_zifeng/article/details/78357898)
* [Beautifying Source Code](https://doc.qt.io/qtcreator/creator-beautifier.html)
* [Clang-Format Style Options](http://clang.llvm.org/docs/ClangFormatStyleOptions.html)
