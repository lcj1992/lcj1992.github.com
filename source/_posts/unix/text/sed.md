---
layout: post
title: sed
date: 2015-12-26
categories: unix
tags: sed
---

http://coolshell.cn/articles/9104.html

#### 概述
`sed  [options]  'command'  file(s)` // 命令行直接执行
`sed  [options]  -f scriptfile  file(s)`　//把命令写到脚本中
流编辑器，与普通的交互式编辑器如vi ，可以交互的接受键盘的命令，进行插入、删除和文本替换等操作。
而流编辑器则是在编辑数据之前，预先指定数据的编辑规则，然后按照规则将数据输出到标准输出。
在流编辑器的所有规则与输入的行匹配完毕后，编辑器读取下一行，重复之前的规则。处理完所有数据后，流编辑器停止。
因此，sed是面向行的，并且sed并不会修改文件本身，除非重定向存储输出。command为具体的文本编辑命令，而file为输入的条件

##### option

|option|说明|
|-|-|
|-n|默认sed处理一行之前会打印输入,-n不打印输入|
|-i|对文件进行修改|
|-e|指定命令(默认可不写)，可组合 -e command1 -e command2，也可-e command1;command2|
|-f|脚本文件|

##### 定址

command中首先要定址，你要告诉sed你希望处理的行,

1.  可以是`数字`，或者`模式`(首次匹配，可多次),区间以逗号分隔`line1,line2`,`line,/pattern/`,`/pattern1/,/pattern2/`
2.  无则处理所有行

#### command

|command|说明|
|-|-|
|`i\`　|匹配行的上一行添加|
|`a\`|匹配行的下一行添加|
|`c\`　|将匹配行替换为目标行|
|`d`|将匹配航删除|
|`s/pattern_old/new/g`|将匹配行替换，`g`处理该行所有匹配，`无`处理第一个，`3g`处理第三个之后的所有(vi中必须指定需要更换的行(`%`所有行)，不指定默认是当前行)|
|`p`|输出|

#### 特殊字符

|字符|说明|
|-|-|
|`^`|匹配行首|
|`$`|匹配行尾|
|`.`|匹配某个字符|
|&|可用来代表匹配的字符|

#### 例子
`sed "s/my/Hao Chen's/g" pets.txt`

匹配pets.txt中所有行，将所有的my替换为Hao chen's

`sed -i "s/my/Hao Chen's/g" pets.txt`

直接对pets.txt进行修改

`sed  's/^/#/g' pets.txt`

在pets.txt的行首添加#

`sed  's/$/ --- /g' pets.txt`

`sed 'a\---' pets.txt`

在pets.txt的行尾添加 ---

`sed 's/<[^>]*>//g' html.txt`

去除html.txt中的标签

`sed "3s/my/your/g" pets.txt`

将pets.txt中的第三行的my替换为your

`sed "3,6s/my/your/g" pets.txt`

将pets.txt中的第三到六行my替换为your

`sed 's/s/S/1' my.txt`

将my.txt中的第一个s替换为S

`sed 's/s/S/3g' my.txt`

将my.txt中的第三个之后的s替换为S

`sed '1,3s/my/your/g; 3,$s/This/That/g' my.txt`

`sed -e '1,3s/my/your/g' -e ' 3,$s/This/That/g' my.txt`

将my.txt中的第一到第三行的my替换为your，并将第三到最后一行的This替换为That

`sed 's/my/[&]/g' my.txt`

将my.txt中的my用中括号括起来。 ***可以用&来当作被匹配的变量***

`sed 'N;s/my/your/g' pets.txt`

将pets.txt中的奇数行的my替换为your，N将下一行的内容纳入缓冲区来进行匹配

`sed 'N;s/\n/,/' pets.txt`

将pets.txt中的奇数行的换行符换成逗号

`sed "1 i This is my monkey,my monkey's name is wukong" my.txt`

在第一行之前插入This..

`sed "1 a This is my monkey,my monkey's name is wukong" my.txt`

在第一行之后插入

`sed "$ a This is my monkey,my monkey's name is wukong" my.txt`

在最后一行之前插入

`sed "/fish/a This is my monkey,my monkey's name is wukong" my.txt`

在匹配到fish的行后插入

`sed "/my/ a --- " my.txt`

在匹配到my的行后插入---

`sed "2 c This is my monkey,my monkey's name is wukong" my.txt`

第二行替换为This...

`sed "/fish/ c This is my monkey,my monkey's name is wukong" my.txt`

将匹配到fish的行，替换为This..

`sed '/fish/ d' my.txt`

将匹配到fish的行，删除/fish/d有没有空格都行

`sed -n '/fish/p' my.txt`

输出匹配到fish的行

`sed '/fish/p' my.txt`
打印输入，并输出匹配到fish的行

`sed -n '/dog/,/fish/p' my.txt`
输出匹配到dog和fish之间的行

`sed -n '1,/fish/p' my.txt`
输出第一行和匹配到fish之间的行

`sed '3,6{/This/{/fish/d}}' pets.txt`
删除第三到六行之间的既包含This，又包含fish的行

`sed '/This/d;s/^ *//g' pets.txt`
`sed '1,${/This/d;s/^ *//g}' pets.txt`
将包含this的行删除，并将行首空格删除

`sed '=' access.log`
显示行号,类似的还有

vi 中输入:set nu 

cat -n xx.log
