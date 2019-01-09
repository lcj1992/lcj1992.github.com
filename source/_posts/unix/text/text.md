---
layout: post
title: 文本
date: 2015-12-24
categories: unix
tags: text tail head　atnodes sort uniq ls cut tee jq sed awk xml2json
---




#### atnodes

    sudo cpan SSH::Batch
    atnodes 'xxcommand'

跳板机免密登陆生产环境机器

    eval `/usr/local/bin/ssh-attach`

偶像agentzh写的哎.

#### cat

cat -n 显示行号,支持通配符*,可以打开多个文件,类似的还有less,more,tail,head,grep

#### echo

echo显示颜色：
echo -e “\033[31m 红色字 \033[0m”

#### tail

显示最后十行

-f 追加显示

-n 8 或者-8   显示八行

    //一般除了看catalina.out和localhost日志,我还会这样看
    tailf bizxx.log | grep ERROR

#### head

显示前十行

-n 8或者-8   显示八行

#### less

追加形式显示

按空格  显示下一页

按F 显示下一屏

按B 显示上一屏

enter　显示下一行

less 比more多的，还可以支持查找 `/pattern`

#### sort

sort -k2 -n (-r)   filename   按第二行排序

-n 数字大小排序

-r　倒序

-t 指定列分割符

-k　按第几个域进行排序，域默认是空格，-t可指定

#### cut

-b  字节

cut -b 3-5（3到5）      cut -b -3（到3）     cut -b 3-（3之后）  cut -b 3  第3    cut -b  3-5,7 （3到5和7）
-c 字符

对于多字节的字符（汉字） -b就呵呵了，此时可以用-nb告诉cut对于多字节的字符不要对其进行拆分处理

-d 指定域分隔符

-f  域

(域分隔符默认是空格，可以通过-d来指定)

`cat /etc/passwd \|head -n 5 \|cut -d : -f 1,3-5`

#### paste

将文本合并,将file2的域追加在file1

paste file1 file2

eg:

文件name:

    1 ligoudan
    2 lierdan
    3 foolchild

文件age:

    1 23
    2 22
    4 24

paste name age

    1 ligoudan	1 23
    2 lierdan	2 22
    3 foolchild	4 foolchild


#### join

将域内容相同的合并，将域内容不同的分别打出(可以类比数据库的join)

Note that join will only work, if there is common field in both file and if values are identical to each other.

join file1 file2

|选项|作用|
|-|-|
|-j|`-j field` 等同于-1 filed -2 field,第一个文件和第二个文件的第field列相同的显示,不指定参数默认是按第一列join(-j 1)|
|-a|`-a filenum` 匹配的行打出,同时将filenum文件中未匹配的行也打出|
|-i|忽略大小写|
|-v|`-v filenum` 类-a,只不过只显示未匹配的行|
|-e|`-e EMPTY` 将需要显示但是文件中不存在的域用此选项指定的字符代替|
|-o|`-o FORMAT` 以指定格式输出|
|-t|`-t CHAR` 以指定字符作为输入输出的分隔符 join 默认以空白字符做分隔符（空格和\t）,可以使用 join -t $'\t'来指定使用tab做分隔符|
|-1|`-1 FIELD` 以file1中FIELD字段进行匹配|
|-2|类上|

ps:

1.  如果只有-1 field1 或者-2 field2,而不是同时存在的,-1 field1 相当于-j field1.
2.  如果1,2,4行都匹配了,也只会打印1,2行, ***断了也就断了***
3.  真的好像数据库的join 比如 `-j 1` 与`select * from file1 join file2 on file1.field1 = file2.field1`  
`-1 1 -2 3`与`select * from file1 join file2 on file1.field1 = file2.field3`,当然还是要谨记2条,  ***断了也就断了***
4.  有时我们需要将多个格式相同的文件join到一起，而join接受的是两个文件的指令，此时我们可以使用管道和字符`-`来实现

     join file1 file2 \| join - file3 \| join - file4  

eg:

    join file1.txt file2.txt
    join -j 1 file1.txt file2.txt
    join -1 2 -2 3 file1.txt file2.txt
    join  -v 1  file1.txt file2.txt

    //-o 指定 将file1的1,2,3列，file2的1,2,3 列都输出。
    //-a指定将file1中不匹配的行也输出，但是file2中没有与file1后两行对应的字段，因此使用empty补齐。
    join -o 1.1 -o 1.2  -o 1.3 -o 2.1 -o 2.2 -o 2.3   -e 'empty' -a 1  file1.txt file2.txt

#### tr

[详见](/2016/04/30/tr)

#### wc

-c (字节)

-m (字符)

-l (lines)

-w  (words)

-L 查看最长的行的长度

#### uniq

**uniq 去重针对的只是连续的两行**

-c  --count

-u 只会显示出现一次的行

-d 只会显示重复出现的行

#### zxx

zcat zless zmore

#### expr
表达式求值 乘法*需要转义`\*`     
`+ - \* %   /`    

#### echo

echo -e 高亮

#### tee

将标准输入复制到文件甚至标准输出例如 ls -al \| tee file.txt

#### grep

[详见](/2015/12/07/grep)

#### awk

[详见](/1025/12/25/awk)

#### sed

[详见](/2015/12/26/sed)

#### jq

    echo 'xxx' \| jq .
    echo 'xxx' \| jq .cabin
    echo 'xxx' \| jq .flight.cabin
    echo 'xxx' \| jq .flight[1].cabin

使用场景：查日志时，你不用从终端中拷出来，然后再格式化了。直接在终端里搞定了，谁用谁知道，爽的不要不要的。

#### xml2json

    echo 'xxx' | xml2json | jq .

#### pbcopy,pbpaste 

大段的文本直接管道到pbcopy,应该会快点的(linux上xclip)

    ls -l | pbcopy
    pbpaste > test


#### 参考 

[1]<http://linuxtools-rst.readthedocs.org/zh_CN/latest/>

[unix下json的命令行工具]<http://blog.chinaunix.net/uid-24774106-id-3830242.html>

[echo显示内容带颜色]<http://www.cnblogs.com/lr-ting/archive/2013/02/28/2936792.html>

[xml2json github]<https://github.com/Inist-CNRS/node-xml2json-command>

[7 command-line tools for data science]<http://jeroenjanssens.com/2013/09/19/seven-command-line-tools-for-data-science.html>

[join]<http://jjuanxi.blog.163.com/blog/static/17527419720121954756361/>
