---
layout: post
title: 享元模式
categories: clean_code
tags: flyweight
---

## 定义


## 例子

菜单上的flavours就是共享的,机场贵宾室说明基本都是共享的,Integer.valueOf()

    // Menu acts as a staticfactory and cache for CoffeeFlavour flyweight objects
    class Menu {
        private Map<String, CoffeeFlavour> flavours = new ConcurrentHashMap<String, CoffeeFlavour>();

        CoffeeFlavour lookup(String flavorName) {
            if (!flavours.containsKey(flavorName))
                flavours.put(flavorName, new CoffeeFlavour(flavorName));
            return flavours.get(flavorName);
        }

        int totalCoffeeFlavoursMade() {
            return flavours.size();
        }
    }

## 优缺点

1. 减少运行时对象实例的个数，节省内存
2. 将许多“虚拟”独享的状态集中管理
3. 一旦你实现了它，那么单个的逻辑实例将无法拥有独立而不同的行为。

## 参考

[wikipedia](https://en.wikipedia.org/wiki/Flyweight_pattern)
