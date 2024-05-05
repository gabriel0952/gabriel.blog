---
layout: post
title: Git Rebase 注意事項
tags: [note, Git]
---

## 什麼是 Rebase
Rebase 從名字可以看出是由 Re 這個字首和 Base 所組合，大概可以有一種去重新修改特定分支的“基本版本”，也就是把另一個分支的版本作為目前分支的基礎 (Base)。

我們可以參考一下 [Lydia Hallie](https://dev.to/lydiahallie) 大大做的 Gif

![Git Rebase](https://res.cloudinary.com/practicaldev/image/fetch/s--EIY4OOcE--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/dwyukhq8yj2xliq4i50e.gif)

## 心得與 Android 使用小工具

在 rebase 的時候有可能會因為修改到程式碼格式(空白、換行等)，或是因為經過大量的 commit 而有相當多的差異，而出現眾多的 conflicts。

這個時候我們可以藉由先把對象merge到身上，然後進行 conflict 的追蹤以及確認。在這一個部份我們可以使用 android studio 裡面的工具進行 conflict 的 review，在 android studio 中 VCS > Git > Resolve Conflicts。開啟後便可以針對有衝突的檔案進行單獨的檢查。

接著在把 merge 發生的 conflict 解決完之後，我們就可以在進行一次 rebase，這一次可能害是會有幾筆的 conflict，不過解決起來應該會比一開始直接做 rebase 來的輕鬆些。同樣在這邊解 conflict 一樣可以。
