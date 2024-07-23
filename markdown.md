markdown 学习笔记

## 常用快捷键

**ctrl + 1 ~6 标题1~6	 	ctrl + B 粗体				ctrl + I _斜体_		<u>下划线：ctrl+u</u>**

**代码块：ctrl+shift+k		插入链接:ctrl+k			分割线：*\*\*		==高亮==  \ctrl+d**

**搜索：ctrl+f				替换：ctrl+h**



* **修改typora默认编程语言**

打开路径文档：/usr/share/typora/resources/appsrc/window

##### 打开frame.js 全局搜索“select a language”，将shell修改为默认语言

("**==select a language==**")+"'></span>",t.childNodes[0].textContent=e||"**==shell==**",t},b=function(e)



## 免费版Typora下载

```
1. windows版本
 
a.百度云-链接: https://pan.baidu.com/s/16GyYOby3VzQ5_7RUaG1bKQ 提取码: 6svf
b.夸克：https://pan.quark.cn/s/2bf3f0bae18d
 
2. mac版本
 
a.百度云-链接: https://pan.baidu.com/s/1XDF8kVi1fAKZRsrKztmZCw 提取码: ucc2
b.夸克：https://pan.quark.cn/s/be320f9e6125
 
3. ubuntu版本
 
a.百度云-链接: https://pan.baidu.com/s/1d4ywfVtsX5dz1DyD6kTi_A 提取码: b5rr
b.夸克：https://pan.quark.cn/s/09b7905a63eb
 
##安装命令：
sudo dpkg -i  ./Typora_Linux_0.11.18_amd64.deb
 
4. 其他linux版本
 
a.百度云-链接: https://pan.baidu.com/s/1pG0mPvxAwod7TPFxt6xiug 提取码: 2551
b.夸克：https://pan.quark.cn/s/e3c3d18fccf1
 
##解压到你安装软件的路径
tar xf typora-0-11-18.tar.gz -C /opt/  
 
##创建桌面文件和图标
vim ~/.local/share/applications/typora.desktop
 
[Desktop Entry]  
Name=Typora  
Comment=A minimal Markdown reading & writing app. Change Log: (https://typora.io/windows/dev_release.html)  
GenericName=Markdown Editor  
Exec=/opt/typora/Typora  
Icon=/opt/typora/resources/assets/icon/icon_256x256@2x.png  
Type=Application  
Categories=Office;WordProcessor;Development;  
MimeType=text/markdown;text/x-markdown;
```



## 标题格式

使用 **#** 号可表示 1-6 级标题，一级标题对应一个 **#** 号，二级标题对应两个 **#** 号，以此类推。

```md
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

显示效果如下

# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题

## 段落

Markdown 段落没有特殊的格式，直接编写文字就好，**段落的换行是使用两个以上空格加上回车**。

如：123  

123

## 字体

Markdown 可以使用以下几种字体：

```md
*斜体文本*
_斜体文本_
**粗体文本**
__粗体文本__
***粗斜体文本***
___粗斜体文本___
```

显示效果如下所示：

*斜体文本*
_斜体文本_
**粗体文本**
__粗体文本__
***粗斜体文本***
___粗斜体文本___

## 分隔线

你可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。你也可以在星号或是减号中间插入空格。下面每种写法都可以建立分隔线：

```
***

* * *

*****

- - -

----------
```

***

* * *

*****

- - -

----------

## 删除线

如果段落上的文字要添加删除线，只需要在文字的两端加上两个波浪线 **~~** 即可，实例如下：

```
RUNOOB.COM
GOOGLE.COM
~~BAIDU.COM~~
```

RUNOOB.COM
GOOGLE.COM
~~BAIDU.COM~~

## 下划线

下划线可以通过 HTML 的 **<u>** 标签来实现：

```
<u>带下划线文本</u>
```

<u>带下划线文本</u>

## 脚注

脚注是对文本的补充说明。

Markdown 脚注的格式如下:

```
[^要注明的文本]
```

以下实例演示了脚注的用法：

```
创建脚注格式类似这样 [^RUNOOB]。

[^RUNOOB]: 菜鸟教程 -- 学的不仅是技术，更是梦想！！！
```

创建脚注格式类似这样 [^RUNOOB]。

[^RUNOOB]: 菜鸟教程 -- 学的不仅是技术，更是梦想！！！

# Markdown 列表

Markdown 支持有序列表和无序列表。

无序列表使用星号(*****)、加号(**+**)或是减号(**-**)作为列表标记，这些标记后面要添加一个空格，然后再填写内容：

```
* 第一项
* 第二项
* 第三项

+ 第一项
+ 第二项
+ 第三项


- 第一项
- 第二项
- 第三项
```

* 第一项
* 第二项
* 第三项

+ 第一项
+ 第二项
+ 第三项


- 第一项
- 第二项
- 第三项

有序列表使用数字并加上 **.** 号来表示，如：

```
1. 第一项
2. 第二项
3. 第三项
```

1. 第一项
2. 第二项
3. 第三项

### 列表嵌套

列表嵌套只需在子列表中的选项前面添加两个或四个空格即可：

```
1. 第一项：
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
2. 第二项：
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素
```

1. 第一项：
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
2. 第二项：
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素

# Markdown 区块

Markdown 区块引用是在段落开头使用 **>** 符号 ，然后后面紧跟一个**空格**符号：

```
> 区块引用
> 菜鸟教程
> 学的不仅是技术更是梦想
```

> 区块引用
> 菜鸟教程
> 学的不仅是技术更是梦想

另外区块是可以嵌套的，一个 **>** 符号是最外层，两个 **>** 符号是第一层嵌套，以此类推：

```
> 最外层
> > 第一层嵌套
> > > 第二层嵌套
```

> 最外层
> > 第一层嵌套
> >
> > > 第二层嵌套

### 区块中使用列表

区块中使用列表实例如下：

```
> 区块中使用列表
> 1. 第一项
> 2. 第二项
> + 第一项
> + 第二项
> + 第三项
```

> 区块中使用列表
> 1. 第一项
> 2. 第二项
> + 第一项
> + 第二项
> + 第三项

### 列表中使用区块

如果要在列表项目内放进区块，那么就需要在 **>** 前添加四个空格的缩进。

列表中使用区块实例如下：

```
* 第一项
    > 菜鸟教程
    > 学的不仅是技术更是梦想
* 第二项
```

* 第一项
    > 菜鸟教程
    > 学的不仅是技术更是梦想
* 第二项

# Markdown 代码

如果是段落上的一个函数或片段的代码可以用反引号把它包起来（**`**），例如：

```md
`printf()` 函数
```

`printf()` 函数

### 代码区块

代码区块使用 **4 个空格**或者一个**制表符（Tab 键）**。

```
tab printf()	
```

​	printf()	

你也可以用 **```** 包裹一段代码，并指定一种语言（也可以不指定）：

```
```javascript
$(document).ready(function () {
    alert('RUNOOB');
});
​```
```

```javascript
$(document).ready(function () {
    alert('RUNOOB');
});
```

# Markdown 链接

链接使用方法如下：

```
[链接名称](链接地址)
```

```
这是一个链接 [菜鸟教程](https://www.runoob.com)
```

这是一个链接 [菜鸟教程](https://www.runoob.com)

直接使用链接地址：

```
<https://www.runoob.com>
```

<https://www.runoob.com

### 高级链接

我们可以通过变量来设置一个链接，变量赋值在文档末尾进行：

```
这个链接用 1 作为网址变量 [Google][1]
这个链接用 runoob 作为网址变量 [Runoob][runoob]
然后在文档的结尾为变量赋值（网址）

  [1]: http://www.google.com/
  [runoob]: http://www.runoob.com/
```

这个链接用 1 作为网址变量 [Google][1]
这个链接用 runoob 作为网址变量 [Runoob][runoob]
然后在文档的结尾为变量赋值（网址）

[1]: http://www.google.com/
[runoob]: http://www.runoob.com/



# Markdown 图片

Markdown 图片语法格式如下：

```
![alt 属性文本](图片地址)

![alt 属性文本](图片地址 "可选标题")
```

- 开头一个感叹号 !
- 接着一个方括号，里面放上图片的替代文字
- 接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上选择性的 'title' 属性的文字。

使用实例：

```
![RUNOOB 图标](https://static.jyshare.com/images/runoob-logo.png)

![RUNOOB 图标](https://static.jyshare.com/images/runoob-logo.png "RUNOOB")
```

![RUNOOB 图标](https://static.jyshare.com/images/runoob-logo.png)

![RUNOOB 图标](https://static.jyshare.com/images/runoob-logo.png "RUNOOB")

当然，你也可以像网址那样对图片网址使用变量:

```
这个链接用 1 作为网址变量 [RUNOOB][1].
然后在文档的结尾为变量赋值（网址）

[1]: https://static.jyshare.com/images/runoob-logo.png
```

这个链接用 1 作为网址变量 [RUNOOB][1].
然后在文档的结尾为变量赋值（网址）

[1]: https://static.jyshare.com/images/runoob-logo.png

Markdown 还没有办法指定图片的高度与宽度，如果你需要的话，你可以使用普通的 <img> 标签。

```
<img src="https://static.jyshare.com/images/runoob-logo.png" width="50%">
```

<img src="https://static.jyshare.com/images/runoob-logo.png" width="50%">

# Markdown 表格

Markdown 制作表格使用 **|** 来分隔不同的单元格，使用 **-** 来分隔表头和其他行。

```
语法格式如下：
|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |

```

| 表头   | 表头   |
| ------ | ------ |
| 单元格 | 单元格 |
| 单元格 | 单元格 |

对齐方式

**我们可以设置表格的对齐方式：**

- **-:** 设置内容和标题栏居右对齐。
- **:-** 设置内容和标题栏居左对齐。
- **:-:** 设置内容和标题栏居中对齐。

实例如下：

```
| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格子 | 单元格子 | 单元格子 |
| 单元格子 | 单元格子 | 单元格子 |
```

| 左对齐   |   右对齐 | 居中对齐 |
| :------- | -------: | :------: |
| 单元格子 | 单元格子 | 单元格子 |
| 单元格子 | 单元格子 | 单元格子 |

### 转义

Markdown 使用了很多特殊符号来表示特定的意义，如果需要显示特定的符号则需要使用转义字符，Markdown 使用反斜杠转义特殊字符：

```
**文本加粗** 
\*\* 正常显示星号 \*\*
```

**文本加粗** 
\*\* 正常显示星号 \*\*

### 公式

**Markdown Preview Enhanced** 使用 [KaTeX](https://github.com/Khan/KaTeX) 或者 [MathJax](https://github.com/mathjax/MathJax) 来渲染数学表达式。

KaTeX 拥有比 MathJax 更快的性能，但是它却少了很多 MathJax 拥有的特性。你可以查看 KaTeX supported functions/symbols 来了解 KaTeX 支持那些符号和函数。

默认下的分隔符：

```
$...$ 或者 \(...\) 中的数学表达式将会在行内显示。
$$...$$ 或者 \[...\] 或者 ```math 中的数学表达式将会在块内显示。
```

```
$$
f(x) = sin(x) +12 
$$
$$ 
\sum_{n=1}^{100} n 
$$
```

$$
\sum_{n=1}^{100} n
$$

$$
f(x) = sin(x) +12
$$

### 1.[typora](https://so.csdn.net/so/search?q=typora&spm=1001.2101.3001.7020)常用快捷键

**ctrl + 1 ~6 标题1~6** 等同于  
一级标题：#+空格+标题内容  
二级标题：##+空格+标题内容  
三级标题：###+空格+标题内容  

**ctrl + B 粗体**

**ctrl + I _斜体_**

**搜索：ctrl+f**

替换：ctrl+h

**代码块：ctrl+alt+f**

<u>**下划线：ctrl+u**</u>

分割线：*\*\*

**插入链接：ctrl+k**

==高亮==  \==内容==

1. **ctrl + / 切换编辑语法模式/预览模式**

2. **ctrl + 1 ~6 标题1~6** 等同于  
    一级标题：#+空格+标题内容  
    二级标题：##+空格+标题内容  
    三级标题：###+空格+标题内容  
    ……  
    最多六级标题

3. **ctrl + B 粗体**

4. **ctrl + I _斜体_**

5. **ctrl + T 表格**

6. **\`\`\`+回车 代码块**

7. > 引用：>+空格

8. [转义字符](https://so.csdn.net/so/search?q=%E8%BD%AC%E4%B9%89%E5%AD%97%E7%AC%A6&spm=1001.2101.3001.7020)： \\ + 字符

- 无序列表：” **\-** “+ 空格 或 “ \* ” + 空格

- 有序列表：数字+“.”+空格

- 生成目录：\[TOC\]按回车

- 选中一整行：ctrl+l

- 选中单词：ctrl+d

- 选中相同格式的文字：ctrl+e

- 跳转到文章开头：ctrl+home

- 跳转到文章结尾：ctrl+end

- **搜索：ctrl+f**

- 替换：ctrl+h

- 引用：输入>之后输入空格

- **代码块：ctrl+alt+f**

- **加粗：ctrl+b**

- 倾斜：ctrl+i

- <u>**下划线：ctrl+u**</u>

- 删除线： ～～内容～～

- 分割线：- - - 或\*\*\*

- **插入链接：ctrl+k**

- 数学公式：$$ + enter

- 表情: :+单词+:

    **常用表情地址：**[https://www.webfx.com/tools/emoji-cheat-sheet/](https://www.webfx.com/tools/emoji-cheat-sheet/)

- 目录：\[toc\] + enter

- 插入图片：直接拖动到指定位置即可或者ctrl+shift+i

**图片可以直接拖进来，但是图片是相对路径，只能在本地使用，如果要放到博客中，要用绝对路径**

**链接可以直接把url复制上去**

| 序号 | 类型                   | 快捷键                             |
| ---- | ---------------------- | ---------------------------------- |
| 1    | 删除线                 | alt+shift+5                        |
| 2    | 下标                   | <sub>内容</sub>                    |
| 3    | 上标                   | <sup>内容</sup>                    |
| 4    | 脚注                   | \[+^+脚注\]                        |
| 5    | ==高亮==               | \==内容==                          |
| 6    | 着重                   | ctrl+shift+\`                      |
| 7    | 淡化                   | +内容+                             |
| 8    | 转义                   | \\ + 符号                          |
| 9    | 清除格式               | ctrl + \\                          |
| 10   | 居中                   | <center>+居中内容+<center>         |
| 11   | 右对齐                 | <p align="right">右对齐</p>        |
| 12   | 左对齐                 | <p align="left">右对齐</p>         |
| 13   | 换行                   | 回车即可或者<br/>                  |
| 14   | 空格（一个汉字俩空格） | (一个空格) (两个空格推荐使用这个） |

### 2.操作

| 序号 | 类型             | 快捷键            |
| ---- | ---------------- | ----------------- |
| 1    | 新建             | Ctrl+N            |
| 2    | 新建窗口         | Ctrl+Shift+N      |
| 3    | 打开             | Ctrl+O            |
| 4    | 快速打开         | Ctrl+P            |
| 5    | 保存             | Ctrl+S            |
| 6    | 另存为           | Ctrl+Shift+S      |
| 7    | 关闭             | Ctrl+W            |
| 8    | 撤销             | Ctrl+Z            |
| 9    | 重做             | Ctrl+Y            |
| 10   | 复制为MarkDown   | Ctrl+Shift+C      |
| 11   | 粘贴为纯文本     | Ctrl+Shift+V      |
| 12   | 选中当前行/句    | Ctrl+L            |
| 13   | 选中当前格式文本 | Ctrl+E            |
| 14   | 选中当前词       | Ctrl+D            |
| 15   | 跳转到文首       | Ctrl+Home         |
| 16   | 跳转到文末       | Ctrl+End          |
| 17   | 查找             | Ctrl+F            |
| 18   | 替换             | Ctrl+H            |
| 19   | 大纲视图         | Ctrl+Shift+1      |
| 20   | 文档列表视图     | Ctrl+Shift+2      |
| 21   | 文件树视图       | Ctrl+Shift+3      |
| 22   | 显示隐藏侧边栏   | Ctrl+Shift+L      |
| 23   | 源代码模式       | Ctrl+/            |
| 24   | 专注模式         | F8                |
| 25   | 打字机模式       | F9                |
| 26   | 切换全屏         | F11               |
| 27   | 实际大小         | Ctrl+Shift+0      |
| 28   | 放大/缩小        | Ctrl + Shift+ +/- |
| 29   | 应用内窗口切换   | Ctrl+Tab          |
| 31   | 打开DevTools     | Shift+F12         |

### 3.使用typora生成pd、html等文件

**文件→导出→选择要导出文件的格式**

### 4.markdown画图

连接：**https://blog.csdn.net/whatday/article/details/88655461**

