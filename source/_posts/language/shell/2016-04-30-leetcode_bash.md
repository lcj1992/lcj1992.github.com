---
layout: post
title: leetcode的脚本全解
categories: language
tags: leetcode shell
---

***[Tenth Line](https://leetcode.com/problems/tenth-line/)***

    ##too too easy
    awk 'NR==10 {print $0}' file.txt

***[Transpose File](https://leetcode.com/problems/transpose-file/)***

    ## memory limit Exceeded了
    col=`awk '{print NF}' file.txt | head -1`; for ((  i=1; i<$col+1; i++ )); do awk -v VAR=$i '{print $VAR}' file.txt | xargs; done

    ## 哎 抄网上的 todo awk脚本
    #fuck
    awk '
    {
        for (i = 1; i <= NF; i++)  {
            a[NR, i] = $i
        }
    }
    NF > p { p = NF }
    END {
        for (j = 1; j <= p; j++) {
            str = a[1, j]
            for (i = 2; i <= NR; i++){
                str = str " " a[i, j]
            }
            print str
        }
    }' file.txt

***[Valid Phone Numbers](https://leetcode.com/problems/valid-phone-numbers/)***

    #grep 正则表达式todo 转义todo,
    #easy
    grep -Eo  '^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$|^[0-9]{3}-[0-9]{3}-[0-9]{4}$' file.txt

***[Word Frequency](https://leetcode.com/problems/word-frequency/)***

    # tr todo,一行一个,然后sort uniq
    # easy
    tr -cs "[:alpha:]" "\n" < words.txt | sort -r | uniq -c | awk '{print $2,$1}' | sort -nrk 2