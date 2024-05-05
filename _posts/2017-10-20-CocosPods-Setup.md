---
layout: post
title: CocoaPods 在 Swift 環境建置與簡單使用
tags: [Swift, CocoaPods]
---

### CocoaPods 是什麼？

CocoaPods 是一種支援 Swift 和 Objective-C 程式開發的第三方庫資源相依性管理工具。藉由 CocoaPods 可以省去逐一將第三方 Class / Framework 加入至專案中的時間。

<!-- more -->

### 在 Mac 電腦安裝 CocoaPods

安裝 CocoaPods 的動作很簡單，前往終端機 (terminal) 並依下列程式碼輸入：

```
sudo gem install cocoapods
```

CocoaPods 是基於 Ruby 語言來開發的，而 Mac 電腦都有自帶 Ruby 系統。所以基本上大多數的 Mac 都可以直接安裝，若是在安裝時出現錯誤也可以按照錯誤訊息做排除，我目前遇到作多次的問題大多是 Ruby 版本上的一些問題導致無法安裝，因此只要對 Ruby 版本作調整/更新，多數狀況可排除。

### 對 Xcode 專案使用 CocoaPods

在 CocoaPods 安裝完成後，就來試試怎麼用吧

首先，建立一個 Xcode 的專案(假設命名為 CocoapodsTest)，之後先把 Xcode 關閉再開啟終端機，使用 `cd` 指令前往 CocoapodsTest 所在的位置

接著我們要在這個專案中加入一個 Podfile 文件，被安裝的 pods 都會寫入至 Podfile 以作紀錄及追踪更新。日後當你呼叫 CocoaPods 去安裝或更新現存的 pods，CocoaPods 便會前往 Podfile 搜找指令。

要建立 Podfile 也很簡單，只需要輸入下面這段指令：

```
sudo gem install cocoapods
```

這樣便會生成 Podfile，像這樣：

```
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'CocoapodsTest' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for CocoapodsTest
end
```

這是 Podfile 的基本結構。接下來我們需要對他做一點點修改，這邊我們需要用到終端機上的一個文字編輯器 Vim ，這邊我們只需要在終端機上下這個指令：

```
vim Podfile
```

假設我們今天要使用的第三方元件只有 `SwiftyJSON` ，那我們只需要把它編輯成配置如下：

```
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'CocoapodsTest' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for CocoapodsTest
  pod 'SwiftyJSON'
end
```

然後按下 `esc` 再輸入 `:wq!` 按下 `Enter` 即可完成編輯。

接著我們只要在完成最後一個步驟就可以將我們需要的第三方資源加入我們的專案中了！
在終端機輸入：

```
pod install
```

他就會開始自動幫你安裝你需要的資源了～

不過要注意的是，在安裝好之後這個專案以後要開啟的話，我們就使用 要使用 `CocoapodsTest.xcworkspace` 而不是 `CocoapodsTest.xcodeproj`。

完成了這些動作之後就可以在專案中像 import UIKit 直接使用了喔！

### CocoaPods 官方網站資源

連結[Link]( https://cocoapods.org/ "Link")
在官網我們可以用我們想要加入專案中的第三方資源名稱作搜尋，來查看現在這個資源適用的版本為何
