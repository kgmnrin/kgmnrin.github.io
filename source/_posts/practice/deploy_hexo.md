---
title: Hexo 部署
date: 2023-06-07
url: deploy_hexo
categories: ["practice"]
tags: ["Hexo", "GitHub"]
references:
	- '[Stellar：开始您全新的博客之旅 - XAOXUU](https://xaoxuu.com/wiki/stellar/#start)'
---

# Deploy Hexo

## Environment

- [Node.js — Run JavaScript Everywhere](https://nodejs.org/en)
- [Git](https://git-scm.com)

```
npm install -g hexo-cli
hexo init
git submodule add https://github.com/xaoxuu/hexo-theme-stellar.git themes/stellar
cd themes/stellar
git checkout tags/1.29.1
npm install hexo-deployer-git --save
```

## Commands

```
hexo g
hexo s
hexo d
```
