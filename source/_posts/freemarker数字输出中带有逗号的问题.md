---
title: freemarker数字输出中带有逗号的问题
date: 2019-07-30 10:21:58
tags:
- freemarker
---

例如：
Long number=1000000L;
model.addAttribute(“number”, number);

$ {number}在freemarker中显示为1,000,000
想显示为1000000，使用${number?c}
加上?c即可
c为freemarker的内建函数
参考资料[https://freemarker.apache.org/docs/ref_builtins_number.html#ref_builtin_c]

