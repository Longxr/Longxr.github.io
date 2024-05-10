---
title: Sublime插件React-Native篇
categories:
  - 环境搭建
tags:
  - Sublime
  - React-Native
date: 2017-04-24 17:38:57
---

<Excerpt in index | 摘要> 
作为一个秒开的编辑器，用了就不想换呀，插件库也很强大，稍作配置就可以用了<!-- more -->
<The rest of contents | 余下全文>

> 作为一个秒开的编辑器，用了就不想换呀，插件库也很强大，稍作配置就可以用了

## 本体
Sublime免费的，只是偶尔弹窗一下,所以别折腾什么破解版了
[官网地址](https://www.sublimetext.com/3),根据你的系统选择下载

## Package Control
安装插件之前要先装下Package Control

### 自动安装
使用Ctrl+`快捷键或者通过View->Show Console菜单打开终端，粘贴如下代码：
```
import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
如果终端安装报错的话，就改用手动安装

### 手动安装
1. 点击`Preferences > Browse Packages`菜单
2. 进入打开的目录的上层目录，然后再进入`Installed Packages/`目录
3. 下载[Package Control.sublime-package](https://packagecontrol.io/Package%20Control.sublime-package)并复制到`Installed Packages/`目录
4. 重启Sublime Text

## 插件安装
### Babel 
> 支持ES6， React.js, jsx代码高亮，对 JavaScript, jQuery 也有很好的扩展

#### 安装
PC：Ctrl+shift+p
Mac：Cmd+shift+p
输入：`pci`，选择Package `Control:install package`
加载出插件列表后，输入babel进行安装

#### 配置
1. 打开.js, .jsx 后缀的文件;
2. 找到菜单栏的view -> Syntax -> Open all with current extension as… -> Babel -> JavaScript (Babel)，选择babel为默认 javascript 打开syntax

### React ES6 Snippets
> 为react语法提示，sublime-react-es6支持es6语法, 另一个类似的插件ReactJS貌似是ES5的，敲出来的代码块方法都有function

#### 安装
PC：Ctrl+shift+p
Mac：Cmd+shift+p
输入：`pci`，选择`Package Control:install package`
加载出插件列表后，输入`React ES6 Snippets`进行安装
#### 支持的代码片段
```
cdm→  componentDidMount: fn() { ... } 
cdup→  componentDidUpdate: fn(pp, ps) { ... } 
cs→  var cx = React.addons.classSet; 
cwm→  componentWillMount: fn() { ... } 
cwr→  componentWillReceiveProps: fn(np) { ... } 
cwu→  componentWillUpdate: fn(np, ns) { ... } 
cwun→  componentWillUnmount: fn() { ... } 
cx→  cx({ ... }) 
fdn→  React.findDOMNode(...) 
fup→  forceUpdate(...) 
gdp→  getDefaultProps: fn() { return {...} } 
gis→  getInitialState: fn() { return {...} } 
ism→  isMounted() 
props→  this.props. 
pt→  propTypes { ... } 
rcc→  component skeleton 
refs→  this.refs. 
ren→  render: fn() { return ... } 
scu→  shouldComponentUpdate: fn(np, ns) { ... } 
sst→  this.setState({ ... }) 
state→  this.state. 
```

### jsfmt
> 格式化JS代码，jsformat也支持jsx格式化，但是在import的地方老是换行挺烦

#### 安装
PC：Ctrl+shift+p
Mac：Cmd+shift+p
输入：`pci`，选择`Package Control:install package`
加载出插件列表后，输入`jsfmt`进行安装
#### 配置
* 添加jsx的相关配置
```
cd ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/jsfmt/
npm i esformatter@latest esformatter-jsx@latest
```

* Preferences -> Package Settings -> Sublime JSFMT，[添加代码](https://github.com/ionutvmi/sublime-jsfmt)：
```
{
  // autoformat on save
  "autoformat": true,

  // array of extensions for autoformat
  "extensions": [
    "js",
    "jsx",
    "sublime-settings"
  ],

  // options for jsfmt
  "options": {
    "preset": "jquery",
    "indent": {
      "value": "    "
    },
    // plugins included
    "plugins": [
      "esformatter-jsx",
    ]
  },

  "jsx": {
    "formatJSX": true,
    // change these to your preferred values
    // refer to https://github.com/royriojas/esformatter-jsx#best-configuration for more options
    "attrsOnSameLineAsTag": false,
    "maxAttrsOnTag": 1,
    "firstAttributeOnSameLine": false,
    "alignWithFirstAttribute": true
  },
  // other esformatter options
  "options-JSON": {
    "plugins": [
      "esformatter-quotes"
    ],
    "quotes": {
      "type": "double"
    }
  },
  "alert-errors": true,
  // path to nodejs
  "node-path": "node",
  // if true it will format the whole file even if you have a selection active
  "ignore-selection": false
}
```

#### 快捷键
打开 Preferences -> Key Bindings - User，添加代码：
`{"keys":["ctrl+q"],"command":"format_javascript"}`

### SublimeLinter-ESLint 
> ESLint 由 JavaScript 红宝书 作者 Nicholas C. Zakas 编写， 2013 年发布第一个版本。 NCZ 的初衷不是重复造一个轮子，而是在实际需求得不到 JSHint 团队响应 的情况下做出的选择：以可扩展、每条规则独立、不内置编码风格为理念编写一个 lint 工具

#### 安装eslint
终端输入：
```
npm install -g eslint@latest 
npm install -g babel-eslint@latest 
npm install -g eslint-plugin-react 
```
验证是否安装成功：
`eslint -v`

#### ESLint配置rules
在home目录(windows下即为：c:\usrs\用户名 )下新建 .eslintrc,添加代码：

```
{
// I want to use babel-eslint for parsing!
"parser": "babel-eslint",
"env": {
 // I write for browser
 "browser": true,
 // in CommonJS
 "node": true
},
// To give you an idea how to override rule options:
"rules": {

    ////////////////
    // 可能的错误  //
    ////////////////
    // 禁止条件表达式中出现赋值操作符
    "no-cond-assign": 2,
    // 禁用 console
    "no-console": 0,
    // 禁止在条件中使用常量表达式
    // if (false) {
    //     doSomethingUnfinished();
    // } //cuowu
    "no-constant-condition": 2,
    // 禁止在正则表达式中使用控制字符 ：new RegExp("\x1f")
    "no-control-regex": 2,
    // 数组和对象键值对最后一个逗号， never参数：不能带末尾的逗号, always参数：必须带末尾的逗号，
    // always-multiline：多行模式必须带逗号，单行模式不能带逗号
    //"comma-dangle": [1, "never"],
    // 禁用 debugger
    "no-debugger": 2,
    // 禁止 function 定义中出现重名参数
    "no-dupe-args": 2,
    // 禁止对象字面量中出现重复的 key
    "no-dupe-keys": 2,
    // 禁止重复的 case 标签
    "no-duplicate-case": 2,
    // 禁止空语句块
    "no-empty": 2,
    // 禁止在正则表达式中使用空字符集 (/^abc[]/)
    "no-empty-character-class": 2,
    // 禁止对 catch 子句的参数重新赋值
    "no-ex-assign": 2,
    // 禁止不必要的布尔转换
    "no-extra-boolean-cast": 2,
    //  禁止不必要的括号 //(a * b) + c;//报错
    "no-extra-parens": 0,
    // 禁止不必要的分号
    "no-extra-semi": 2,
    // 禁止对 function 声明重新赋值
    "no-func-assign": 2,
    //  禁止在嵌套的块中出现 function 或 var 声明
    "no-inner-declarations": [2, "functions"],
    // 禁止 RegExp 构造函数中无效的正则表达式字符串
    "no-invalid-regexp": 2,
    // 禁止在字符串和注释之外不规则的空白
    "no-irregular-whitespace": 2,
    // 禁止在 in 表达式中出现否定的左操作数
    "no-negated-in-lhs": 2,
    //   禁止把全局对象 (Math 和 JSON) 作为函数调用  错误：var math = Math();
    "no-obj-calls": 2,
    // 禁止直接使用 Object.prototypes 的内置属性
    "no-prototype-builtins":0,
    // 禁止正则表达式字面量中出现多个空格
    "no-regex-spaces": 2,
    // 禁用稀疏数组
    "no-sparse-arrays": 2,
    // 禁止出现令人困惑的多行表达式
    "no-unexpected-multiline": 2,
    // 禁止在return、throw、continue 和 break语句之后出现不可达代码
    /*
        function foo() {
        return true;
        console.log("done");
        }//错误
    */
    "no-unreachable": 2,
    // 要求使用 isNaN() 检查 NaN
    "use-isnan": 2,
    // 强制使用有效的 JSDoc 注释
    "valid-jsdoc": 1,
    // 强制 typeof 表达式与有效的字符串进行比较
    // typeof foo === "undefimed" 错误
    "valid-typeof": 2,

    //////////////
    // 风格指南  //
    //////////////

     //"quotes": [2, "single"],
     "eol-last": [0],
     "no-mixed-requires": [0],
     "no-underscore-dangle": [0],

     //////////////
    // ES6.相关 //
    //////////////

    //"arrow-spacing":[2,{ "before": true, "after": true }],
    // 强制在子类构造函数中用super()调用父类构造函数，TypeScrip的编译器也会提示
    "constructor-super": 0,
    // 强制 generator 函数中 * 号周围使用一致的空格
    "generator-star-spacing": [2, { "before": true, "after": true }],
    // 禁止修改类声明的变量
    "no-class-assign":2,
    // 不允许箭头功能，在那里他们可以混淆的比较
    "no-confusing-arrow":0,
    // 禁止修改 const 声明的变量
    "no-const-assign":2,
    // 禁止类成员中出现重复的名称
    "no-dupe-class-members":2,
    // 不允许复制模块的进口
    "no-duplicate-imports":0,
    // 禁止 Symbol  的构造函数
    "no-new-symbol":2,
    // 允许指定模块加载时的进口
    "no-restricted-imports":0,
    // 禁止在构造函数中，在调用 super() 之前使用 this 或 super
    "no-this-before-super": 2,
    // 禁止不必要的计算性能键对象的文字
    "no-useless-computed-key":0,
    // 要求使用 let 或 const 而不是 var
    "no-var": 0,
    // 要求或禁止对象字面量中方法和属性使用简写语法
    "object-shorthand": 0,
    // 要求使用箭头函数作为回调
    "prefer-arrow-callback":0,
    // 要求使用 const 声明那些声明后不再被修改的变量
    "prefer-const": 0,
    // 要求在合适的地方使用 Reflect 方法
    "prefer-reflect":0,
    // 要求使用扩展运算符而非 .apply()
    "prefer-spread":0,
    // 要求使用模板字面量而非字符串连接
    "prefer-template":0,
    // Suggest using the rest parameters instead of arguments
    "prefer-rest-params":0,
    // 要求generator 函数内有 yield
    "require-yield":0,
    // enforce spacing between rest and spread operators and their expressions
    "rest-spread-spacing":0,
    // 强制模块内的 import 排序
    "sort-imports":0,
    // 要求或禁止模板字符串中的嵌入表达式周围空格的使用
    "template-curly-spacing":1,
    // 强制在 yield* 表达式中 * 周围使用空格
    "yield-star-spacing":2

},
"plugins":[
"react"
]
}
```
#### Sublime配置
安装插件
PC：Ctrl+shift+p
Mac：Cmd+shift+p
输入：`pci`，选择`Package Control:install package`
输入`SublimeLinter`进行安装
输入`SublimeLinter-contrib-eslint`进行安装

#### default path设置 
Preferences -> Package Settings -> Sublime Linter.sublime-settings 
，添加代码：
```
"paths": { 
            "osx": [ 
                "/usr/local/lib" 
            ] 
        }, 
        "python_paths": { 
            "osx": [ 
                "/usr/local/lib" 
            ] 
        }, 
```


## 参考文章
[Sublime Text3关于react的插件——react语法提示&代码格式化](http://www.jianshu.com/p/ecf6c802fdc5)

[Sublime Text 3 搭建 React.js 开发环境](https://segmentfault.com/a/1190000003698071)

[详解 ESLint 规则，规范你的代码](http://blog.guowenfh.com/2016/08/07/ESLint-Rules/)

[Sublime text3 jsx支持（eslint）](http://www.jianshu.com/p/b2985ba08ec9)

