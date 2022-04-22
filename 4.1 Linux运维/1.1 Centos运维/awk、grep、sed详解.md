[awk、grep、sed详解](https://www.cnblogs.com/GaoLou/p/14923088.html)

## **概述**

awk、grep、sed 是 linux 操作文本的三大利器，也是必须掌握的 linux 命令之一。

三者的功能都是处理文本，但侧重点各不相同，其中属 awk 功能最强大，但也最复杂。grep 更适合单纯的查找或匹配文本，sed 更适合编辑匹配到的文本，awk 更适合格式化文本，对文本进行较复杂格式处理。

简单概括：

- grep：数据查找定位
- awk：数据切片
- sed：数据修改

## **grep = global regular expression print**

用最简单术语来说，grep(全局正则表达式打印)--命令用于查找文件里符合条件的字符串。 从文件的第一行开始，grep 将一行复制到  buffer 中，将其与搜索字符串进行比较，如果比较通过，则将该行打印到屏幕上。grep将重复这个过程，直到文件搜索所有行。

 注意：这里没有进程执行 grep 存储行、更改行或仅搜索部分行。

### **示例数据文件**

请将以下数据剪切粘贴到一个名为 “sampler.log” 的文件中:

1. `boot`
2. `book`
3. `booze`
4. `machine`
5. `boots`
6. `bungie`
7. `bark`
8. `aardvark`
9. `broken$tuff`
10. `robots`

### **一个简单例子**

grep 最简单的例子是:

1. `grep "boo" sampler.log `

在本例中，grep 将遍历文件 “sampler.log” 的每一行，并打印出其中的每一行 包含单词“boo”:

1. `boot`
2. `book`
3. `booze`
4. `boots`

但是如果你操作的是大型文件，就会出现这种情况：如果这些行标识了文件中的哪一行，它们是什么，可能对你更有用，如果需要在编辑器中打开文件，那么可以更容易地跟踪特定字符串做一些改变。 这时候可以通过添加 -n 参数来实现:

1. `grep -n "boo" sampler.log`

这产生了一个更有用的结果，解释了哪些行与搜索字符串匹配:

1. `1:boot`
2. `2:book`
3. `3:booze`
4. `5:boots`

另一个有趣的参数是 -v，它会打印出相反的结果。换句话说，grep 将打印所有与搜索字符串不匹配的行，而不是打印与之匹配的行。 

在下列情况下，grep 将打印不包含字符串 “boo” 的每一行，并显示行号，如上一个例子所示

1. `grep -vn "boo" sampler.log `
2. `4:machine`
3. `6:bungie`
4. `7:bark`
5. `8:aardvark`
6. `9:broken$tuff`
7. `10:robots`

c 选项告诉 grep 抑制匹配行的打印，只显示匹配行的数量，匹配查询的行。 例如，下面将打印数字4，因为有4个在 `sampler.log` 中出现 “boo”。

1. `grep -c "boo" sampler.log `
2. `4`

l 选项只打印查询中具有与搜索匹配行的文件的文件名字符串。 如果你想在多个文件中搜索相同的字符串，这将非常有用。像这样:

1. `grep -l "boo" *`

对于搜索非代码文件，一个更有用的选项是 -i，忽略大小写。这个选项将处理在匹配搜索字符串时，大小写相等。 在下面的例子中，即使搜索字符串是大写的，包含“boo”的行也会被打印出来。

1. `grep -i "BOO" sampler.log `
2. `boot`
3. `book`
4. `booze`
5. `boots`

x 选项只精确匹配。换句话说，以下命令搜索没有结果，因为没有一行只包含"boo"

1. `grep -x "boo" sampler.log `

最后，-A 允许你指定额外的上下文件行，这样就得到了搜索字符串额外行，例如

1. `grep -A2 "mach" sampler.log`
2. `machine`
3. `boots`
4. `bungie`

### **正则表达式**

正则表达式是描述文本中复杂模式的一种紧凑方式。

有了 grep 你可以使用搜索模式( pattern ) 。其他工具使用正则表达式 (regexp) 以复杂的方式。而 grep  使用的普通字符串，实际上非常简单正则表达式。如果您使用通配符，如 ' * ' 或 ' ? '，比如列出文件名等，你可以使用 grep  使用基本的正则表达式进行搜索 

例如搜索文件以字母 e 结尾的行:

1. `grep "e$" sampler.log `
2. `booze`
3. `machine`
4. `bungie`

如果需要更广泛的正则表达式命令，则必须使用 `grep-E`。 

例如，正则表达式命令 ? 将匹配1或0次出现 之前的字符:

1. `grep -E "boots?" sampler.log `
2. `boot`
3. `boots`

你还可以使用 `pipe(|)` 结合多个搜索，它的意思是 “或者”，所以你可以这样做:

1. `grep -E "boot|boots" sampler.log `
2. `boot`
3. `boots`

### **特殊字符**

如果你想搜索的是一个特殊字符，该怎么办?如果你想找到所有的直线，如果包含美元字符“$”，则不能执行 `grep“$”a_file`，因为 '$' 将被解释为正则表达式，相反，你将得到所有的行，其中有任何作为行结束，即所有行。 解决方案是 “转义” 符号，所以您将使用

1. `grep '\$' sampler.log `
2. `broken$tuff`

你还可以使用 “-F” 选项，它代表“固定字符串”或“快速”，因为它只搜索字符串，而不是正则表达式。

### **更多的 regexp 的例子**

参考：http://gnosis.cx/publish/programming/regular_expressions.html

## **AWK**

由 Aho，Weinberger 和 Kernighan 创建的文本模式扫描和处理语言。 

AWK非常复杂，所以这不是一个完整的指南，但应该给你一个知道什么 awk 可以做。它使用起来比较简单，强烈建议使用。

### **AWK 基础知识**

awk 程序对输入文件的每一行进行操作。它可以有一个可选的 BEGIN{ } 部分在处理文件的任何内容之前执行的命令，然后主{ }部分运行在文件的每一行中，最后还有一个可选的END{ }部分操作将在后面执行文件读取完成:

1. `BEGIN { …. initialization awk commands …}`
2. `{ …. awk commands for each line of the file…}`
3. `END { …. finalization awk commands …}`

对于输入文件的每一行，它会查看是否有任何模式匹配指令，在这种情况下它仅在与该模式匹配的行上运行，否则它在所有行上运行。 这些 `'pattern-matching'` 命令可以包含与 grep 一样的正则表达式。 

awk 命令可以做一些非常复杂的数学和字符串操作，awk也支持关联阵列。 AWK 将每条线视为由多个字段组成，每个字段由“间隔符”分隔。 默认情况下，这是一个或多个空格字符，因此行：

1. `this is a line of text`

包含6个字段。在 awk 中，第一个字段称为 $1，第二个字段称为 $2，等等，全部行称为 $0。 

字段分隔符由 awk 内部变量 FS 设置，因此如果您设置 FS= ": "则它将根据 ':' 的位置划分一行，这对于 `/etc/passwd` 之类的文件很有用，其他有用的内部变量是 NR，即当前记录号(即行号) NF是当前行中字段的数量。 

AWK 可以对任何文件进行操作，包括 `std-in`，在这种情况下，它通常与 '|' 命令一起使用，例如，结合 grep 或其他命令。 

例如，如果我列出当前目录中的所有文件

1. `ls -l`
2. `总用量 140`
3. `-rw-r--r-- 1 root root 55121 1月  3 17:03 combined_log_format.log`
4. `-rw-r--r-- 1 root root 80644 1月  3 17:03 combined_log_format_w_resp_time.log`
5. `-rw-r--r-- 1 root root   71 1月  3 17:55 sampler.log`

` 我可以看到文件大小报告为3 列数据。如果我想知道它们的总大小，这个目录中的文件我可以做:

1. `ls -l | awk 'BEGIN {sum=0} {sum=sum+$5} END {print sum}'`
2. `135836`

请注意，'print sum' 打印变量 sum 的值，因此如果 sum = 2 则 'print sum' 给出输出 '2' 而 'print $ sum' 将打印 '1' ，因为第二个字段包含值 '1' 。 

因此，会很简单编写一个可以计算平均值的和一列数字的标准偏差的 awk 命令 - 在主要内部积累 'sumx' 和 'sumx2' 部分，然后使用标准公式计算 END 部分的平均值和标准偏差。 

AWK 支持（'for' 和 'while'）循环和分支（使用 'if '）。 所以，如果你想修剪一个文件并且只在每个第 3 行操作，你可以这样做：

1. `ls -l | awk '{for (i=1;i<3;i++) {getline}; print NR,$0}'`
2. `3 -rw-r--r-- 1 root root 80644 1月  3 17:03 combined_log_format_w_resp_time.log`
3. `4 -rw-r--r-- 1 root root   71 1月  3 17:55 sampler.log`

for 循环使用 “getline” 命令遍历文件，并且每隔3次才打印一行。

注意，由于文件的行数是4，不能被3整除，所以最后一个�打�