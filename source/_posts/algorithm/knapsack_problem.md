---
layout: post
title: 背包问题
date: 2014-11-10
categories: algorithm
tags: knapsack dp
---

### 01_knapsack

    #include <stdio.h>

    int max(int m,int n){
      return m > n ? m :n;
    }

    int main(){
      int v[4] = {12,10,20,15};
      int w[4] = {2,1,3,2};
      int n = 4;
      int W = 5;

      int m[5][5];
      int i,j;
      // 都没选择物品，总价值肯定为0啊
      for( i = 0; i < W;i++){
          m[0][i] = 0;
      }

      // 物品一个一个选
      for(i = 1; i <= n;i++ ){
          for(j = 0; j <=W;j++){
              // 物品个数为i，重量最大为j的情况下，总价值最大多少？

              if(w[i -1] > j){
              // 如果第i个物品的重量大于总重量j，则不能选物品i，即在总重量最大允许为j的前提下，选i个物品的总价值等于选i-1的总价值
                  m[i][j] = m[i-1][j];
              } else {
              // 否则的话，比较总重量最大值为j的前提下，选物品i和不选物品i的总价值最大值。
                  m[i][j] = max(m[i -1][j],m[i-1][j-w[i-1]] + v[i-1]);
              }
              printf("%d,%d,%d\n",i,j,m[i][j]);
          }
      }
      return 0;
    }

### 参考 

[1]<https://en.wikipedia.org/wiki/Knapsack_problem>
