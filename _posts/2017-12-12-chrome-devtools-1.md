---
layout: post
title: Chrome DevTools 紀錄 (一)
abstract: 介紹「.cls, designMode, 隱藏 element」工具的用法
comments: true
---

## 多個 class 的效果比較

可以將想要比較的 styles 分成多個 class，然後使用 devtools 裡面的 .cls 切換不同 class 進行比較。

![class_toggle]({{ Site.baseurl }}/public/images/class_toggle.png){:height="460px"}

<br>

## 測試內文工具

有時會需要測試如果版面的文字長一點的話會不會導致破版，一般都是
從 HTML 的 source code 修改，使用上比較麻煩。

可以在 console 中輸入以下：

    document.designMode = "on";

接著就可以直接在頁面上打字。

<br>

## 隱藏元素 (element)

開發中有時會需要查看某個 HTML element 隱藏時的效果如何，這時可以直接選取該 element，然後按下 `h`，這樣就可以隱藏，再按一次就會出現。

> 按下後會自動加上 `visibility: hidden !important`，因此該 element 所佔的空間還是會存在。

