---
title: RPhoto - JavaEE簡易相冊管理系統
date: 2013-06-08 19:31:17
tags:
  - Java
  - 個人項目
---


![20130606113154.png](http://7b1fa0.com1.z0.glb.clouddn.com/old-blog/20130606113154.png)

這是一個用Java EE實現的簡易的相冊管理系統。這是我的Java EE課程設計的項目，完成時間很短，所以整個項目還不完善，後台寫得比較混亂，也有部分BUG和來不及實現的功能。不過即便如此，在兩三天的時間內，把這個項目完成，並且設計了一個良好的界面，還是覺得挺有成就的。畢竟這是我的第一個Java EE項目，初學者總是被Java EE的“龐大”嚇倒了。我也覺得Java EE上手可沒PHP那麽快，學習JSP、Servlet、JavaBean、自定義標簽等還是需要花挺多時間的。

Github項目主頁：https://github.com/Raysmond/RPhoto

<!-- more -->

## 功能模塊
2. 相冊管理
 新建專輯、上傳照片、查看照片、查看幻燈片
3. 評論
對照片進行評論
4. 標簽
給照片添加標簽
5. 用戶系統
6. 後台管理
用戶管理、相冊管理、評論管理、標簽管理和統計等

## 所用技術
- JSP+Servlet+JavaBean+MySQL
- JSP+jQuery/Ajax+Servlet+JavaBean+MySQL
- JSP+自定義標簽+JavaBean+MySQL
- 一個前端HTML\CSS框架：IVORY
http://weice.in/ivory/
https://github.com/kanthvallampati/IVORY
- 一個多圖無刷新上傳插件：[Uploadify](http://www.uploadify.com/)
用uploadify可以在不刷新頁面的情況下上傳多張照片，在PHP和JSP都可以用，很方便。
- 一個幻燈片插件：[Galleria](http://galleria.io/)

總之，這個算是Java EE入門項目，給以給初學者借鑒借鑒，同時可以看看如何在JavaEE在不用框架技術的情況下，如何寫出一個較爲完整的網站。

截圖
![login.png](http://7b1fa0.com1.z0.glb.clouddn.com/old-blog/login.png)

![20130606113154.png](http://7b1fa0.com1.z0.glb.clouddn.com/old-blog/20130606113154.png)

![20130606115802.png](http://7b1fa0.com1.z0.glb.clouddn.com/old-blog/20130606115802.png)
