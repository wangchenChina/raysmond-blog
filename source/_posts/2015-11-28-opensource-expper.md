---
title: Expper 文章收藏和分享網站的源碼開源了
date: 2015-11-28 01:11:06
tags:
  - Spring
  - JHipster
  - 開源
  - 個人項目
---

![http://7b1fa0.com1.z0.glb.clouddn.com/0001.png](http://7b1fa0.com1.z0.glb.clouddn.com/0001.png)

網站地址：https://www.expper.com
Github 地址：https://github.com/Raysmond/expper

我目前是一名研究生，最近在學習 Spring 框架和分布式系統開發，出于學習和分享的目的，我決定把Expper的源碼開源了。我個人非常喜歡和支持開源，它在我的學習道路上幫助我了太多。 Expper 是一個文章收藏和分享的網站，開源的目的是和大家分享我的代碼和學習成果，也希望開源能夠幫助 Expper 社區發展的更好。

<!-- more -->

## Expper 是一個怎樣的網站？

一句話來說， Expper 是一個文章收藏工具和分享社區。有下面這些 features:

### 文章收藏

- 結合 Chrome 插件，一鍵保存網絡文章（類似 pocket ）
- 雲端保存文章，簡潔優雅的文章格式和排版
- 高效整理和搜索文章



### 文章分享

- 分享和交流各個技術領域的文章
- 只會展示文章標題和摘要和原文連接， expper 絕不會公開全文轉載原文
- 通過不同的話題和標簽歸類整理技術文章
- 所有話題和標簽文章具有熱度排序和時間排序功能

### 技術棧

- 最重要的是： Spring Boot 和 JHipster
- 數據庫： PostgreSQL
- 緩存和統計： Redis
- 消息隊列： RabbitMQ
- 前端： Grunt + Angularjs + Bootstrap + SASS

Spring Boot 和 JHipster 太好用了，對于我這個學習 Spring 不到半年的人來說，是一個很好的起點。另外加上 Redis 和 RabbitMQ ，網站性能提升了好多，速度還挺滿意。

### 截圖

![http://7b1fa0.com1.z0.glb.clouddn.com/0002.png](http://7b1fa0.com1.z0.glb.clouddn.com/0002.png)


——-
歡迎注冊體驗，https://www.expper.com
歡迎前往 Github 項目主頁圍觀， star 或者 contribute ：https://github.com/Raysmond/expper
