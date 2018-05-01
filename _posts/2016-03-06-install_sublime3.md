---
layout: post
title: 安装sublime3
excerpt: "安装sublime3"
modified: 2016-03-06
tags: [程序]
comments: true
image:
  feature: sample-image-4.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---


下载地址：[链接](https://www.sublimetext.com/3)


## 安装Package Control
地址：[链接](https://packagecontrol.io/installation)
或者ctrl+`,在输入框里输入
```
import urllib.request,os,hashlib; h = '2915d1851351e5ee549c20394736b442' + '8bc59f460fa1548d1514676163dafc88'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
## 安装包含大量配色主题的插件包
ctrl+shift+p：Colorsublime Plugin

### 安装Material Theme
- ctrl+shift+p:install
- Material Theme

git：[地址](https://github.com/equinusocio/material-theme)
配置如下：

```javascript
    // Default
    "theme": "Material-Theme.sublime-theme",
    "color_scheme": "Packages/Material Theme/schemes/Material-Theme.tmTheme",
    
    // Darker
    // "theme": "Material-Theme-Darker.sublime-theme",
    // "color_scheme": "Packages/Material Theme/schemes/Material-Theme-Darker.tmTheme",
    
    // Lighter
    // "theme": "Material-Theme-Lighter.sublime-theme",
    // "color_scheme": "Packages/Material Theme/schemes/Material-Theme-Lighter.tmTheme",
```

### 安装Soda主题
- ctrl+shift+p:install
- Theme - Soda

配置如下：
下面没有改动配色

```json
{
    "theme": "Soda Light 3.sublime-theme"
}

{
    "theme": "Soda Dark 3.sublime-theme"
}
```

## 安装markdown环境
- MarkdownEditing
- Markdown Extended
- Markdown Preview

如果你喜欢 Soda Dark 和 Monokai，我建议你使用 Monokai Extended (GitHub)。这个 color scheme 是 Monokai Soda 的增强，如果再配合 Markdown Extended (GitHub)，将大大改善 Markdown 的语法高亮。

## 输入框跟随
安装IMESupport

## 转换为UTF-8编码
ConvertToUTF8

## 格式化HTML/CSS/JS代码
HTMLPrettify

## 高亮显示配对括号
BracketHighlighter

## 自动补全代码
SublimeCodeIntel 


For Windows:

- Jump to definition = ``Alt+Click``
- Jump to definition = ``Control+Windows+Alt+Up``
- Go back = ``Control+Windows+Alt+Left``
- Manual CodeIntel = ``Control+Shift+space``
##自动对齐
Alignment
ctrl+alt+a就可以使其按照等号对其。

## 格式化代码

Astyle

## 格式转换
pandoc
首先安装pandoc，[链接](http://pandoc.org/installing.html)
修改Pandoc.sublime-settings：

```JSON
"pandoc-path": "C:/Program Files (x86)/Pandoc/pandoc.exe",
```
下载miniTex,[链接](http://miktex.org/download)


## HTML和CSS代码编辑插件

Emmet


## 快捷键
- Ctrl + Enter：在当前行下面新增一行然后跳至该行

- Ctrl + Shift + Enter：在当前行上面增加一行并跳至该行

- Ctrl + ←/→：进行逐词移动

- Ctrl + Shift + ←/→进行逐词选择

- Ctrl + ↑/↓移动当前显示区域

- Ctrl + Shift + ↑/↓移动当前行

- Alt + F3：选中当前关键字出现的所有位置

- Ctrl + Shift + F：多文件搜索&替换

- Ctrl + P：跳转到指定文件，输入文件名后可以

- Ctrl + F/H：进行标准查找/替换，之后：

- Alt + C：切换大小写敏感（Case-sensitive）模式

- Alt + W：切换整字匹配（Whole matching）模式

- Alt + R：切换正则匹配（Regex matching）模式

- Ctrl + Shift + H：替换当前关键字

- Ctrl + Alt + Enter：替换所有关键字匹配

- F3：跳至当前关键字下一个位置
- Shift + F11：切换无干扰全屏

- Alt + Shift + 2：进行左右分屏

- 分屏之后，使用Ctrl + 数字键跳转到指定屏，使用Ctrl + Shift + 数字键将当前屏移动到指定屏

- 使用Ctrl + N在当前窗口创建一个新标签，Ctrl + W关闭当前标签，Ctrl + Shift + T恢复刚刚关闭的标签。
- 编辑代码时我们经常会开多个窗口，所以分屏很重要。Alt + Shift + 2进行左右分屏，Alt + Shift + 8进行上下分屏，Alt + Shift + 5进行上下左右分屏（即分为四屏）。

- Ctrl + P快速跳转到文件夹里的文件

- 使用Ctrl + Shift + N创建一个新窗口（该快捷键再次和搜狗输入法快捷键冲突，个人建议禁用所有搜狗输入法快捷键）。
当窗口内没有标签时，使用Ctrl + W关闭该窗口。Shift + Ctrl + T快速恢复关闭的标签

- Shift + F11切换无干扰全屏

- Ctrl + [向左缩进，Ctrl + ]向右缩进，此外Ctrl + Shift + V可以以当前缩进粘贴代码（非常实用）。
- Ctrl + Shift + V进行粘贴，可以在粘贴的过程中保持缩进，这时格式都是正确的。
##设置setting

- shift+↓ 向下选中多行。

- Shift+← 向左选中文本。
- shift+↑ 向上选中多行。


- Shift+→ 向右选中文本。
```JSON
// 设置Sans-serif（无衬线）等宽字体，以便阅读
"font_face": "YaHei Consolas Hybrid", 
"font_size": 12, 
// 使光标闪动更加柔和
"caret_style": "phase", 
// 高亮当前行
"highlight_line": true, 
// 高亮有修改的标签
"highlight_modified_tabs": true,

良好实践（Good Practices）良好的代码应该是规范的，所以Google为每一门主流语言都设置了其代码规范（Code Style Guideline）。我自己通过下面的设置使以规范化自己的代码。
// 设置tab的大小为2"tab_size": 2,
// 使用空格代替tab"translate_tabs_to_spaces": true,
// 添加行宽标尺"rulers": [80, 100],
// 显示空白字符"draw_white_space": "all",
// 保存时自动去除行末空白"trim_trailing_white_space_on_save": true,
// 保存时自动增加文件末尾换行"ensure_newline_at_eof_on_save": true,

```

## 安装SublimeLinter

  SublimeLinter是Sublime的一个代码检测工具插件。
安装SublimeLinter前必须安装node.js

打开Sublime，按下 Ctrl+Shift+p 进入 Command Palette;
输入install进入 Package Control: Install Package;
输入SublimeLinter，选择SublimeLinter进行安装。

npm install htmlhint -g

## 安装jshint

npm install htmlhint -g

1.npm config set registry https://registry.npm.taobao.org 
2.npm info underscore 








