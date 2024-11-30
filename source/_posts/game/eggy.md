---
title: 蛋仔派对
date: 2024-09-16
categories: ["game"]
tags: ["Eggy", "NetEase"]
---

# Eggy

Eggy 项目中踩过的一些坑和其他遇到的东西。

## Neox

## Cocos

## C++

## Python

- 兼容性问题，安装环境时需看全下载 Python 2.718，而不是 Python 2.7。
- Python 2 遍历字典时需使用 .iteritems()，range() 使用 xrange() 代替。
- 使用 / 做除法时，如果明确返回的需要是个 int，需要用 //。
- for 循环里用 lambda 闭包时，对象改变时闭包调用依然时为最终值，可以通过改成函数参数来替代。

## Co-worker
