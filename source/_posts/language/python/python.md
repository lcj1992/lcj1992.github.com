---
layout: post
title: python basic
date: 2014-11-07
categories:
    - python
---

**数据类型**：

*	空          `None`  
*   布尔类型     `Boolean`  
*	整型        `Int`  
*	浮点型      `Float`  
*	字符串 `String`
*	列表 `List`
*	元组 `Tuple`
*	集合 `Set`
*	字典 `Dict`
*	日期 `Date`

**控制流**：

if语句：

	  #!/usr/bin/python
	  number = 23
	  guess  = int(raw_input('Enter an integer:'))

	  if guess == number:
	          print 'equals'
	          print 'but you do not win any prizes!'
	  elif guess > number:
	          print 'greater'
	  else:
	          print 'smaller'
	  print 'Done'

while语句：（python 中循环可以和else连用）

	  #!/usr/bin/python
	  number = 23
	  running = True

	  while running:
	          guess = int(raw_input('Enter an integer:'))

	          if guess ==  number:
	                  print 'Congratulations'
	                  print 'but you do not win any prizes.'
	                  running = False
	          elif guess < number:
	                  print 'lower than that'
	                  print 'try again'
	          else:
	                  print 'higher than that'
	                  print 'try again'
	  else:
	          print 'The while loop is over.'
	  print 'Done.'

for循环

	  #!/usr/bin/python

	  for i in range(1, 5):
	      print i
	  else:
	      print 'The for loop is over'

break语句：

	  #!/usr/bin/python

	  while True:
	          s = raw_input('Enter something:')
	          if s == 'quit':
	                  break
	          print 'Length of the string is', len(s)
	  print 'Done'

continue语句：

	  #!/usr/bin/python
	  while True:
	          s = raw_input('Enter something:')
	          if s == 'quit':
	             continue
	          print 'Length of the string is', len(s)
	  print 'Done'

`from xxx import yyy`和`import yyy`的区别与java静态导入类似，前者引用xxx直接引用，后者需要加yyy前缀yyy.xxx

python的良好的编程习惯

*	该加空格时要加
*	没有花括号，依靠对齐来区分代码层次逻辑，所以同一层次的代码要对齐，不然报错。


ps:

`python -m SimpleHTTPServer 7777` 把当前目录当做web服务根目录,端口是7777, 电脑手机发送文件利器

`sudo yum install MySQL-python`