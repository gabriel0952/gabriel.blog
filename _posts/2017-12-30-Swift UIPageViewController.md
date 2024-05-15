---
layout: post
title: 用 UIPageViewController 來製作屬於APP的使用說明(引導頁面)
tags: [iOS, Swift, UIPageViewController]
categories: [iOS]
---

前言
===

相信在使用 APP 都有曾看過在第一次打開一個 app 的時候跳出一個關於 app 的使用引導，雖然我自己好像幾乎都是一直略過居多，不過剛好段時間老師提出要加這個功能進到他自己的 app ，所以也嘗試了一些方式，最早是嘗試用了 UIScrollView

不過若是使用 UIScrollView 來做導覽頁功能的話，其實有點像是暴力硬解，藉由自訂 ScrollView 的長寬，在切出一個一個頁面的方式來實作，雖然能夠完成需要的功能不過實在有點不方便，因此近期在重構的時候也順便多研究了一下新的方法 —— UIPageViewController

<!-- more -->

Part1：屬於 APP 的使用說明
===

首先一如往常建置一個 Swift APP 專案，然後打開 main.storyboard ，這邊可以把原本預設的 viewcontroller 砍掉

再來，我們可以從 object lirary 裡拉一個 Page View Controller 出來，然後在 attributes inspector 裡面將它設為 initial view controller ，接著再拉一個 view controller 出來當作我們後續用來顯示引導頁面的內容頁，然後照自己的需求放進想放的東西

這邊我由上到下放了一個 'Button, Label, Label, image view, text view, page control' 佈局也是看個人需要自行調整，如下圖工作區域

![](https://i.imgur.com/o0ZqZnP.png)

接著，我們再新增兩個 swift 檔案，一個繼承自 UIViewController，一個繼承自 UIPageViewController
首先，繼承自 UIViewController 的檔案是用來後續要顯示每頁內容時所需的控制
另外，繼承自 UIPageViewController 的檔案是用來進行一些在 page view controller 上的工作(例如：上頁、下頁，內容指派等等)








