---
layout: post
title: 责任链模式
categories: clean_code
tags: chain_of_responsibility
---

### 定义

### 实例

logic执行框架

链式，流程需分步进行，且各步之间有层次关系，1执行完成执行2，2执行完成执行3

![类图](/images/design_pattern/chain_of_responsibility.png)

抽象handler、各handler子类，handler都持有下一步骤handler的引用。

类图中着重看标红处，各子类需持有继承者successor的引用，如果满足条件，执行successor的方法。

```
abstract class PurchasePower {
    static final double BASE = 500;
    PurchasePower successor;

    void setSuccessor(PurchasePower successor) {
        this.successor = successor;
    }
    abstract public void processRequest(PurchaseRequest request);
}
```

[代码示例](https://github.com/lcj1992/learn/blob/master/java/designPattern/src/main/java/behavioral/chainOfResponsibility/ChainOfResponseTest.java)

### 参考
[wikipedia](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)
