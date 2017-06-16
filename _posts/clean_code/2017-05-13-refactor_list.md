---
layout: post
title: 重构
categories: clean_code
tags: clean_code
---

* TOC
{:toc}

#### 开篇的例子

开篇介绍了一个例子：

这是一个影片出租店用的程序，计算每一位顾客的消费金额并打印详单，操作者告诉程序：顾客租了哪些影片，租期多长，横须便根据租赁时间和影片类型计算出费用。影片分为三类：普通片，儿童片和新片。除了计算费用，还要为常客计算积分，积分会根据租片种类为新片而有不同。

建议读这本书的，一定要跟着这个例子走一下。看看大师是怎么一步步的重构的。


* 重构的第一步永远相同，为即将修改的代码建立一组可靠的测试环境，这些***测试必须有自我检验能力，好的测试时重构的根本***。

* 代码块越小，代码的功能就越容易管理，代码的处理和移动也就越轻松

* 临时变量的作用： 收集器、计数、解释、复用、元素（详见实现模式笔记）

* 临时变量往往引发问题，它们只在所属的函数中有效，所以它们会助长冗长而复杂的函数。(它接受each.getCharge()的执行结果，然后就不再有任何改变，所以我可以运用replace temp with query把它除去)

* 观察amountFor()时，我发现这个函数使用了来自Rental类的信息，却没有使用来自Customer类的信息（***逻辑中的所有数据都来自另一个类，这时可以运用move method，将这段逻辑搬到另一个类中，逻辑要和数据在一起***)。

* 最好不要在另一个对象的属性基础上运用switch语句，如果不得不使用，也应该在对象自己的数据上使用，而不是在别人的数据上使用，原理同上条.

* 计算费用时需要两项数据：租期长度和影片类型，为什么我选择将租期长度传给movie对象，而不是将影片类型传给rental对象呢？因为本系统可能发生的变化时加入新影片类型，这种变化带有不稳定倾向，如果影片类型有所变化，我希望尽可能控制它所造成的影响，所以选择在movie对象内计算费用。(**某段逻辑需要来自不同对象的数据，那么看较易引起逻辑变化的数据来自哪个对象，就把这段逻辑放在那个对象所属的类，尽可能控制变化所造成的影响**)

* 重构时不要担心整体性能，优化时才需要担心它们。

* 重构的一项重要技巧就是“寻找边界条件”

#### 重构原则

##### 何谓重构

1. 对软件内部结构的一种调整，目的是在***不改变软件可观察行为的前提***下，***提高其可理解性***，***降低其修改成本***。

2. 使用一系列重构手法，在不改变软件可观察行为的前提下，调整其结构

3. 两顶帽子：使用重构技术开发软件时，你把自己的时间分配成两种截然不同的行为：添加新功能，以及重构。无论何时，你应该清楚自己戴的是哪一顶帽子。

##### 为何重构

1. 重构改进软件设计：重构就是在整理代码，经常性的重构可以帮助代码维持自己该有的形态。

2. 重构帮助找到bug：对代码的理解，可以帮助找到bug

3. 重构提高编程速度：改善设计，提高可读性，减少错误，良好的设计是快速开发的根本。

##### 何时重构

1. 重构应该随时随地的进行。

2. 添加功能时重构
    * 是否能对这段代码进行重构，使我能更快地理解它
    * 代码的设计无法帮助我轻松的添加我所需要的特性，如果用某种方式来设计，添加特性会简单很多，这种情况下我也会进行重构。

3. 修补错误时重构

4. 复审代码时重构

##### 怎么对经理说

很多经理嘴巴上说自己“质量驱动”，其实更多是“进度驱动”，这种情况下我会给他们一个较有争议的建议：不要告诉经理（我觉得挺好），受进度驱动的经理要我尽可能快速完事，至于怎么完成，那就是我的事了，我认为最快的方式就是重构，所以我重构。

1. 计算机科学是这样一门科学：它相信所有问题都可以通过增加一个间接层来解决。 ------Kent Beck

2. 间接层是一柄双刃剑，每次都把一个东西分成两份，你就需要多管理一个东西。如果某个对象委托另一个对象，后者又委托另一对象，程序会愈加难以阅读。基于这个观点，你会希望尽量减少间接层。别急，伙计！间接层有它的价值：
    * 允许逻辑共享，比如说一个子函数在两个不同的地点被调用，或超类中的某个函数被所有子类共享
    * 分开解释意图和实现
    * 隔离变化
    * 封装条件逻辑

##### 重构的难题

1. 数据库
2. 修改接口： 如果重构手法改变了已发布接口，你必须同时维护新旧两个接口，直到所有用户都要时间对这个变化做出反应，幸运的是，这不太困难，你通常都有办法把事情组织好，让旧接口继续工作，请尽量这么做：让旧接口调用新接口。

#### 代码的坏味道

##### duplicated code (重复代码)

1. extract method : 同一个类的两个函数含有相同的表达式

2. pull up method : 两个互为兄弟的子类内含有相同表达式

3. from template method : 两个互为兄弟的子类内含有相同表达式

4. substitute algorithm : 有两个函数以不同的算法做着相同的事，可以选择其中一个较清晰的一个，将其他函数的算法替换掉

5. extract class : 如果两个毫不相关的类出现重复代码

##### long method (过长函数)

1. extract method : 百分之九十九的场合里

2. replace temp with query ：如果函数内有大量临时变量

3. introduce parameter object ：如果函数有大量的参数

4. perserve whole object（保持对象完整） ：如果函数有大量的参数

5. replace method with method object : 

6. decompose conditional： 条件表达式是提炼的信号

7. 循环也是提炼的信号。

##### Large class (过大的类)

1. extract class： 

2. extract subclass：

3. duplicate observed data： 

##### long parameter list(过长参数列表)

1. replace parameter with method：如果向已有的对象发出一条请求就可以取代一个参数

2. preserve whole object：将来自同一个对象的一堆数据收集起来，并以该对象替换它们

3. introduce parameter object： 如果某些收却反合理的对象归属，可以为他们制造一个“参数对象”

4. 有时候你明显不希望造成“被调用对象”与“较大对象”之间的某种依赖关系，这时候将数据从对象中拆解出来单独作为参数也是合情合理的。

##### divergent change (发散式变化)

1. extact class ：如果某个类经常因为不同的原因在不同的方向上发生变化

##### shotgun surgey （霰弹式修改）

1. move method、move field ：如果每遇到某种变化，你都必须在许多不同的类内做出许多小修改

divergent change是指“一个类受多种变化的影响”

shotgun surgey是指“一种变化引发多个类相应修改”

##### feature ency （依恋情结）

1. move method : 函数对某个类的兴趣高过对自己所处类的兴趣

2. extract method : 

##### data clumps （数据泥团）

1. extract class ：你常常可以在很多地方看到相同的三四项数据：两个类中相同的字段，许多函数签名中相同的参数

2. introduce parameter object：

3. preserve whole object ：

一个好的评判办法是： 删除众多数据中的一项。这么做，其他数据并没有因而失去意义？如果他们不再有意义，这就是个明确的信号：你应该为它们产生一个新对象。（变化率）

##### primitive obsession（基本类型偏执）

1. replace data value with object: 将原本单独存在的数据值替换成对象

2. replace type code with class : 如果想要替换的数据值是类型码，而它并不影响行为

3. replace type code with subclass: 如果你有与类型码相关的条件表达式

4. replace type code with state/strategy: 同上

5. extract class: 如果有一组总是被发在一起的字段

6. introduce parameter object: 如果你在参数列表中看到基本数据

7. replace array with object: 如果你发现自己正从数组中挑选数据

##### switch statements （switch 语句）

1. extract method 

2. replace type code with subclass

3. replace type code with state/strategy

4. replace conditional with polymorphism

5. replace parameter with explicit methods

6. introduce null object

##### parallel inheritance hierarchies (平行继承体系)

1. move method

2. move field

shotgun surgery的特殊情况

##### lazy class （冗赘类）

1. collapse hierarchy

2. inline class

##### speculative generality （夸夸其谈未来性）

1. collapse hirerchy

2. iniline class

3. remove parameter

4. rename method


##### temporary field (令人迷惑的暂时字段)

1. extract class

2. introduce null object

##### message chains（过度耦合的消息链）

1. hide delegate

2. extract method

3. move method

##### middle man (中间人)

1. remove middle man

2. inline method

3. replace delegation with inheritance

##### inappropriate intimacy (狎昵关系)

1. move method

2. move field

3. change bidirectional association to unidirectional

4. extract class

5. hide delegate

6. replace inheritance with delegation

##### alternative classes with different interfaces

1. rename method

2. move method

3. extract superclass

##### incomplete library （不完美的库类）

1. move method

2. introduce foreign method

3. introduce local extension

##### data class （纯稚的数据类）

1. encapsulate field

2. encapsulate collection

3. remove setting method

4. move method

5. extract method

6. hide method

##### refused bequest （被拒绝的遗赠）

1. push down method

2. push down field

3. replace inheritance with delegation

#### comments （过多的注释）

1. extract method

2. rename method

3. introduce assertion


#### 

#### 重构列表


#### 重新组织函数
Add Parameter（添加参数） 275

Change Bidirectional Association to Unidirectional（将双向关联改为单向） 200

Change Reference to Value（将引用对象改为实值对象） 183

Change Unidirectional Association to Bidirectional（将单向关联改为双向） 197

Change Value to Reference（将实值对象改为引用对象） 179

Collapse Hierarchy（折叠继承体系） 344

Consolidate Conditional Expression（合并条件式） 240

Consolidate Duplicate Conditional Fragments（合并重复的条件片段） 243

Convert Procedural Design to Objects（将过程化设计转化为对象设计） 368

Decompose Conditional（分解条件式） 238

Duplicate Observed Data（复制「被监视数据」） 189

Encapsulate Collection（封装群集） 208

Encapsulate Downcast（封装「向下转型」动作） 308

Encapsulate Field（封装值域） 206

Extract Class（提炼类） 149

Extract Hierarchy（提炼继承体系） 375

Extract Interface（提炼接口） 341

Extract Method（提炼函数） 110

Extract Subclass（提炼子类） 330

Extract Superclass（提炼超类） 336

Form Template Method（塑造模板函数） 345

Hide Delegate（隐藏「委托关系」） 157

Hide Method（隐藏某个函数） 303

Inline Class（将类内联化） 154

Inline Method（将函数内联化） 117

Inline Temp（将临时变量内联化） 119

Introduce Assertion（引入断言） 267

Introduce Explaining Variable（引入解释性变量） 124

Introduce Foreign Method（引入外加函数） 162

Introduce local Extension（引入本地扩展） 164

Introduce Null Object（引入 Null 对象） 260

Introduce Parameter Object（引入参数对象） 295

Move Field（搬移值域） 146

Move Method（搬移函数） 142

Parameterize Method（令函数携带参数） 283

Preserve Whole Object（保持对象完整） 288

Add Parameter（添加参数） 275

Change Bidirectional Association to Unidirectional（将双向关联改为单向） 200

Change Reference to Value（将引用对象改为实值对象） 183

Change Unidirectional Association to Bidirectional（将单向关联改为双向） 197

Change Value to Reference（将实值对象改为引用对象） 179

Collapse Hierarchy（折迭继承体系） 344

Consolidate Conditional Expression（合并条件式） 240

Consolidate Duplicate Conditional Fragments（合并重复的条件片段） 243

Convert Procedural Design to Objects（将过程化设计转化为对象设计） 368

Decompose Conditional（分解条件式） 238

Duplicate Observed Data（复制「被监视数据」） 189

Encapsulate Collection（封装群集） 208

Encapsulate Downcast（封装「向㆘转型」动作） 308

Encapsulate Field（封装值域） 206

Extract Class（提炼类） 149

Extract Hierarchy（提炼继承体系） 375

Extract Interface（提炼接口） 341

Extract Method（提炼函数） 110

Extract Subclass（提炼子类） 330

Extract Superclass（提炼超类） 336

Form Template Method（塑造模板函数） 345

Hide Delegate（隐藏「委托关系」） 157

Hide Method（隐藏某个函数） 303

Inline Class（将类内联化） 154

Inline Method（将函数内联化） 117

Inline Temp（将临时变量内联化） 119

Introduce Assertion（引入断言） 267

Introduce Explaining Variable（引入解释性变量） 124

Introduce Foreign Method（引入外加函数） 162

Introduce local Extension（引入本㆞扩展） 164

Introduce Null Object（引入 Null 对象） 260

Introduce Parameter Object（引入参数对象） 295

Move Field（搬移值域） 146

Move Method（搬移函数） 142

Parameterize Method（令函数携带参数） 283

Preserve Whole Object（保持对象完整） 288

Refactoring – Improving the Design of Existing Code

Pull Up Constructor Body（构造函数本体㆖移） 325

Pull Up Field（值域㆖拉） 320

Pull Up Method（函数㆖拉） 322

Push Down Field（值域㆘移） 329

Push Down Method（函数㆘移） 328

Remove Assignments to Parameters（移除对参数的赋值动作） 131

Remove Control Flag（移除控制标记） 245

Remove Middle Man（移除㆗间㆟） 160

Remove Parameter（移除参数） 277

Remove Setting Method（移除设值函数） 300

Rename Method（重新命名函数） 273

Replace Array with Object（以对象取代数组） 186

Replace Conditional with Polymorphism（以多态取代条件式） 255

Replace Constructor with Factory Method（以工厂函数取代构造函数） 304

Replace Data Value with Object（以对象取代数据值） 175

Replace Delegation with Inheritance（以继承取代委托） 355

Replace Error Code with Exception（以异常取代错误码） 310

Replace Exception with Test（以测试取代异常） 315

Replace Inheritance with Delegation（以委托取代继承） 352

Replace Magic Number with Symbolic Constant（以字面常量取代魔法数） 204

Replace Method with Method Object（以函数对象取代函数） 135

Replace Nested Conditional with Guard Clauses（以卫语句取代嵌套条件式） 250

Replace Parameter with Explicit Methods（以明确函数取代参数） 285

Replace Parameter with Method（以函数取代参数） 292

Replace Record with Data Class（以数据类取代记录） 217

Replace Subclass with Fields（以值域取代子类） 232

Replace Temp With Query（以查询取代临时变量） 120

Replace Type Code with Class（以类取代型别码） 218

Replace Type Code with State/Strategy（以 State/Strategy 取代型别码） 227

Replace Type Code with Subclasses（以子类取代型别码） 223

Self Encapsulate Field（自封装值域） 171

Separate Domain from Presentation（将领域和表述/显示分离） 370

Separate Query from Modifier（将查询函数和修改函数分离） 279

Split Temporary Variable（剖解临时变量） 128

Substitute Algorithm（替换你的算法） 139

Tease Apart Inheritance（梳理并分解继承体系）

[重构](https://zacharychang.gitbooks.io/refactoring-improving-the-design-of-existing-code/)
