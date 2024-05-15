---
layout: post
title: 自己做自己的口罩查詢APP (1) - HttpURLConnection資料我全都要
tags: [Android, Java]
categories: [Android]
---

## 前言
因為工作需要所以重新學習以經好幾年沒有碰的 Android 開發，還記得大學的時候對 Android 開發印象真的是差到爆表...，所以後來也就都沒有在自己去玩它了，不過既然工作有需要硬著頭皮還是得把它學下去 QQ

順便也藉由這次的機會好好的練習一下自己"紀錄"的能力，不定期地將學習和練習成果更新上來分享，也希望得到有用的建議~

-----

## 概述
這次剛好因為口罩之亂，所以想說乾脆試著做一個可以查詢口罩資訊的簡單APP來當作練習，納在這個練習中我們會用到的主功能有：

1. 與[政府資料開放平台所][link1]提供的資料做串接
2. 將下載回來的CSV檔做解析，取得我們想要的內容
3. 簡單的畫面配置
4. [ListView][link2]的使用
5. [SharedPreferences][link3]的使用

剩下可能還有一點其他的小功能，後面若是有用到就另外在附上。
不過主要的會用到的功能基本都在這邊，那麼我們就開始吧。

-----

## 由網站取得需要資料
我們先看一下由政府資料開放平台所提供的資料長得如何吧~
![https://ithelp.ithome.com.tw/upload/images/20200324/20125739ir770Q2sGl.png](https://ithelp.ithome.com.tw/upload/images/20200324/20125739ir770Q2sGl.png)

其中我們可以看到它的欄位說明有說道，它包含了 `醫事機構代碼、醫事機構名稱、醫事機構地址、醫事機構電話、成人口罩剩餘數、兒童口罩剩餘數、來源資料時間` 這些欄位。

那我們現在該如何從這個網站取得我們需要的資料呢？
那就是基本的使用 HttpURLConnection 搭配 GET 來項開放平台索取我們需要的資料~

### 網路存取權限
那這邊我們首先要注意的一點是，因為我們使用網路去取得資料的這一個動作勢必會需要使用用戶的網路資源，因此我們會需要在 `AndroidManifest.xml` 中加入使用網路的權限需求，如果沒有加的話程式會直接閃退喔~

> AndroidManifest.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.example....">
   <application
       android:allowBackup="true"
       ...
   </application>
   <uses-permission android:name="android.permission.INTERNET" />
</manifest>
```

OK 在完成了一點點的前置作業後，我們就可以開始在我們的 MainActivity.java 中加入我們主要的動作了。

### HttpURLConnection 連結目的地

這邊我們先讓這個 APP 可以取得我們想的資料就好，所以我們在 `onCreate` 中加入以下程式碼。

> MainActivity.java

``` java
URL url = new URL("https://data.nhi.gov.tw/Datasets/Download.ashx?rid=A21030000I-D50001-001&l=https://data.nhi.gov.tw/resource/mask/maskdata.csv");
HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
httpURLConnection.setRequestMethod("GET");
httpURLConnection.connect();
```

### 例外處理

在你把這段程式碼放進去的時候，你應該會發現在有些定方會出現紅色的蚯蚓，提醒你這些地方有點問題，那是甚麼問題呢？我們可以把滑鼠移到他們的上面就可以看見 `unhandled exception java.net.malformedurlexception`這個問題，其實這個只是他要你去處理可能發生的例外狀況，那我們可以把這整段使用一組 try catch 來處理他們的例外狀況。

那加上這段例外處理後，我們的程式碼就會變成下面這樣：

> MainActivity.java

``` java
try {
    URL url = new URL("https://data.nhi.gov.tw/Datasets/Download.ashx?rid=A21030000I-D50001-001&l=https://data.nhi.gov.tw/resource/mask/maskdata.csv");
    HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
    httpURLConnection.setRequestMethod("GET");
    httpURLConnection.connect();
} catch (IOException e) {
    e.printStackTrace();
}
```

到了這邊其實基本的連線已經快差不多了，要是你有手癢讓他跑下去看看結果如何的話，你應該會發現....會閃退...，不過別擔心其實這不是你的問題，只是因為 Android 希望我們把網路存取的動作移到背景的 thread 中去執行，不要放在主執行續去處理這些有可能厚實很久的動作。

因此我們只需要再加上一個 thread 來處理這個聯網的動作就好了~

> MainActivity.java

``` java
new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            URL url = new URL("https://data.nhi.gov.tw/Datasets/Download.ashx?rid=A21030000I-D50001-001&l=https://data.nhi.gov.tw/resource/mask/maskdata.csv");
            HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
            httpURLConnection.setRequestMethod("GET");
            httpURLConnection.connect();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}).start();
```

### 資料處理與顯示
好了～到這邊我們取得資料的動作也基本大功告成！
蛤？你說資料呢？其實他們已經都載下來了喔，如果想要顯示出來看的話，我們就加上一個資料再處理的function就好了。

> MainActivity.java

``` java
private void showDownload(HttpURLConnection httpURLConnection) throws IOException {
    InputStream inputStream = httpURLConnection.getInputStream();
    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));

    try {
        String csvLine;
        while ((csvLine = reader.readLine()) != null) {
            String[] row = csvLine.split(",");
            for (int i = 0; i < row.length; i++) {
                System.out.print(row[i]);
            }
            System.out.println("");
        }
    } catch (IOException ex) {
        throw new RuntimeException("Error in reading CSV file: " + ex);
    } finally {
        try {
            inputStream.close();
        } catch (IOException e) {
            throw new RuntimeException("Error while closing input stream: " + e);
        }
    }
}
```

這部分程式碼後半段一樣是在處理例外事件，而前段的 while 便是進行我們資料取得後的要做的事情。
這邊稍微解釋一下 **httpURLConnection.getInputStream();** 是以 getInputStream 的方法獲取由 server 回傳的輸入流，然後指派給 inputStream。接著再用 BufferedReader 來做讀取，最後把讀取的物件放進 while 中針對每一行，一行一行的處理。處理的過程中，因為 CSV 欄位之間是使用 "," 做區隔，因此我們也需要針對 "," 做切割，完成後的結果我們這邊先使用 `System.out.print(row[i]);` 來展示一下即可，未來我們再針對我們的需求做調整。

最後我們稍微看一下輸出的結果吧。
![https://ithelp.ithome.com.tw/upload/images/20200324/201257399jvIyMoywc.png](https://ithelp.ithome.com.tw/upload/images/20200324/201257399jvIyMoywc.png)

[link1]: https://data.gov.tw/
[link2]: https://developer.android.com/reference/android/widget/ListView
[link3]: https://developer.android.com/reference/android/content/SharedPreferences
[link4]: https://github.com/gabriel0952/HttpURLConnectionTest/tree/master
