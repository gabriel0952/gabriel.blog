---
layout: post
title: 在 iOS APP 加上 SQLite
tags: [Swift, SQLit]
---

## SQLite 介紹
### 在 iOS app中的資料儲存

在一個完整的 app 裡面通常都會需要資料的儲存，或是資料傳遞的控制，要做到這些功能有許多方式，而其中較為常見的莫過於就是使用 `SQLit` ，在Swift中也有其他方式能夠達成，像是如果只需要儲存少量資訊的時候可以使用 `NSUserDefaults` ，而如果是對 SQLit 沒有經驗的朋友，也可以嘗試另一個工具 `Core Data` 來實作資料庫的功能。

<!-- more -->

### SQLite

SQLit，是一個輕量的關聯式資料庫管理系統( Relational Database Management System ，縮寫為 RDBMS )，所有的資料內容其實就是一個檔案，而且絕大部分的 SQL 指令都可以使用。

後面會用到一些常見的 SQLite 指令，所以要是對 SQLite 沒有概念可以稍微看一下相關的文件更快上手的。


## 環境建置
### 加入SQLite

在 iOS app 上會出現許多資料儲存方式有個原因可能是，SQLite 的環境建置其來有點小費工，因為其實 Swift 本身是無法直接使用 SQLite 的功能，必須要利用 Objective-C (在 Swift 之前用來編寫 iOS 應用程式的程式語言)來與 SQLite 連結。

首先先建立一個專案，名字就隨自己喜歡創一個吧，我在這邊命名為 mySqliteEx。
接下來介紹加入SQLite的步驟吧：

預設的環境是不會加入 SQLite的，所以這邊需要我們手動加入，先找到`TARGETS > mySqliteEx > General > Linked Frameworks and Libraries`，並按下加號`+`

這邊會列出來所有可以加入的函式庫，所以在上面搜尋填入sqlite來搜尋，最後會出現兩個類似的函式庫，實際上兩個指的是一樣的東西，所以選一個加入就好，我們選libsqlite3.tbd，按下Add加入

![](https://i.imgur.com/7oyuZTd.png)

如果有成功加入的話，就會出現在列表裡面

![](https://i.imgur.com/bIKpaNq.png)

### 新增標頭檔
前面有說到， 因為需要利用 Objective-C 來與 SQLite 連結，所以必須要在應用程式加上一個 header 檔案以連結。

首先，先以新增檔案來加入一個標頭檔，並且命名為 `BridgeHeader.h` 。

接著我們打開這個檔案，然後在這支檔案裡面加上一行程式，如下：

```swift
#include "sqlite3.h"
```

這個 header 檔案是用來引入 sqlite 函式庫，所以填寫這一行程式即可。

### 與 Objective-C 連結
最後我們需要把我們的 Swift 與前一步設置好的 header 檔連結起來，首先找到 `TARGETS > mySqliteEx > Build Settings > Objective-C Bridging Header` ，這邊有很多東西可以設定，所以你可以在右上角的搜尋框輸入bridg來過濾

接著對 `Objective-C Bridging Header` `mySqliteEx` 下的欄位點兩下，會彈出一個輸入框，填入 header 檔案的路徑與名稱(請記得路徑也要填)

成功的話應該會長這樣：

![](https://i.imgur.com/nFM5Kt6.png)

到這個地方其實我們的 SQLite 已經建置完畢，所以我們可以 `Bulid` 一下專案，如過建置成功就代表真的成功了喔！

## 開始使用 SQLite 
### 宣告SQLite變數
接著我們終於來到程式碼的部分，首先會需要宣告一個變數來儲存 SQLite 的連線資訊，型別為COpaquePointer，後續的資料庫操作都會需要這個變數：

```swift
var db :OpaquePointer? = nil
```

### 建立資料庫檔案路徑
SQLite 其實可以視為是一個檔案的操作，所以我們要先取得這個資料庫檔案的路徑：

```swift
// 資料庫檔案的路徑
let urls = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)
let sqlitePath = urls[urls.count-1].absoluteString + "sqlite3.db"
```

這邊會取得此應用程式在手機裡的 Document 目錄，而sqlite3.db則是這個資料庫檔案名稱，這邊你也可以設為自己想要的名稱。

而當程式執行到這邊發現目錄中沒有這麼檔案的話，它會自動建立一個名稱為此的檔案。

### 與資料庫連線
```swift
if sqlite3_open(sqlitePath, &db) == SQLITE_OK {
    print("Successfully opened database")
} else {
    print("Unable to open database.")
}
```

### 建立資料表
