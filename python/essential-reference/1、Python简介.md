# 1.1 运行Python

Python程序是由解释器来执行的。通常在shell中输入`python`即可启动解释器：

![](img/chap1/img0.png)

Python的交互模式是它最有用的功能。你可以输入人以合法的语句或语句序列，然后立即看到结果。

> 交互模式中，特殊变量`_`保存最后一次运算的结果。

执行Python程序：

```shell
python hello.py
```

在UNIX中，可以在程序首行使用`#!`，如：

```python
#!/usr/bin/env python
print "Hello World"
```

# 1.2 变量和算术表达式

Python是动态类型的语言，在程序执行的过程中，变量名会被绑定到不同的值，而且这些值可以属于不同的类型。

变量名是无类型的，在执行过程中可以引用任意类型的数据。

在同一行上，使用分号来隔开多条语句。

Python 不会指定缩进的空格数，只要在一个代码块中保持一致就行了。通常4个空格最常见。

`format`函数使用的格式说明符类似于传统格式化运算符(`%`)使用的说明符。

```python
print "%3d %0.2f" % (year, principal)
print format(year, "3d"), format(principal, "0.2f")
print "{0:3d} {1:0.2f}".format(year, principal)
```
"{0:3d} {1:0.2f}" 冒号前的数字代表传递给`format`的相关参数，冒号后是格式说明符。

# 1.3 条件语句

```python
if a < b:
    pass # do nothing
else:
    print "No"
```

使用`or`、`and`和`not`关键字，可以组成布尔表达式。

行尾使用反斜杠`\`，可以另起一行继续写代码。

Python没有`switch`或`case`来检测多个值，只能借助：

```python
if suffix == ".html":
	content = "text/html"
elif suffix = ".jpg":
    content = "img/jpeg"
else:
    raise RuntimeError("Unknown content type")
```

`in`操作符常用于检查某个值是否包含在另一对象（字符串、列表、字典）中。

# 1.4 文件输入输出

```python
f = open("foo.txt")
line = f.readline()
while line:
    print line, # 后面跟,将忽略换行符
    line = f.readline()
f.close()
```

等同于：

```python
for line in open("foo.txt"):
    print line,
```

写入文件，要在`print`语句后使用`>>`指定一个文件（Python 2）：

```python
f = open("out", "w")
while yead <= numyears:
    print >>f, "%3d %0.2f" % (year, principal)
    year += 1
f.close()    
```

`>>`语法只能用于 Python 2，如果使用Python 3，可将`print`改写如下：

```python
print ("3d %0.2f" % (year, principal), file = f)
```

另外，文件对象支持`write()`方法写入原始数据，上述`print`等同于：

```python
f.write("3d %0.2f\n" % (year, principal))
```

如果要交互式读取用户使用，可以从文件`sys.stdin`读取。要将数据输出到屏幕上，可以写入`sys.stdout`中。

```python
import sys
sys.stdout.write("Enter your name: ")
name = sys.stdin.readline()
```

