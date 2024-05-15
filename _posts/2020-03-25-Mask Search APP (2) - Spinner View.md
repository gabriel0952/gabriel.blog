---
layout: post
title: 自己做自己的口罩查詢APP (2) - 打開吧 我的 Spinner View 
tags: [Android, Java]
categories: [Android]
---

## 回顧

我們在上一篇的內容中，使用了 httpURLConnection 與政府資料開放平台所提供的口罩資訊資料做連結來取得我們所需的 CSV 檔，並且也在最後藉由簡單的處理將下載下來的資料處理成我們需要的格式。

所以說在完成了上一篇內容的工作後我們已經有了內容，那接下來我們要做的事情就是該如何把這些內容表示成我們需要的樣子。當然我們可以用最最最簡單的方式把資料直接塞進一個 TextView 之中，你說這樣內容太多怎麼辦？那就再把 TextView 塞到 ScrollView 裡面啊。

其實這樣的方式也不是不行，但是要做成一個比較完整的 APP 總是不能這麼的隨便啊，所以在這邊我們先試著使用 ListView 來呈現與口罩相關的內容。

那我預計在主要的顯示畫面中可以顯示 `醫事機構名稱、醫事機構地址、醫事機構電話、成人口罩剩餘數、兒童口罩剩餘數、來源資料時間`，並且可以的話讓使用者可以去篩選它想要看到的縣市，並且可以有一個刷新的按鈕讓使用者可以在他想要的時間去刷新。

大概的外觀就會長成這樣~
![https://ithelp.ithome.com.tw/upload/images/20200325/20125739MFNm8yjNSl.png](https://ithelp.ithome.com.tw/upload/images/20200325/20125739MFNm8yjNSl.png)

-----

## 開工
在我們有了想法之後就馬上開始著手吧~
為了避免內容過長，所以這一篇會著重在一些元件的配置上，以及 Spinner 自定義的部分。

### 前置作業

首先我們可以先將一些後續會用到的縣市資料、圖示等等的前置作業先導入到我們的專案中，我在這邊會用到刷新的圖示，所以課已先把它加入 drawable 的資料夾中(可以在 Project模式下 > app > main > res 新增一個 drawable-nodpi 的資料夾來存放會用到的 .png)

並且在後續作縣市篩選的時候會用到台灣各縣市的資訊，因此為了後續方便我們也先做點小處理，在 value 中我們新增一個 string2.xml 的檔案，其中以 string-array 的型態存放我們所有的縣市名稱。

> string2.xml
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string-array name="city">
        <item>臺北市</item>
        <item>新北市</item>
        ...
        <item>嘉義市</item>
    </string-array>
</resources>
```

### 畫面設定

畫面的設計跟排版這部分其實可以交由各位自行設定，不一定要完全一樣要是有想增減的也可自行調整，這邊就提供一個範例。

![https://ithelp.ithome.com.tw/upload/images/20200325/20125739fsy3NScD6l.png](https://ithelp.ithome.com.tw/upload/images/20200325/20125739fsy3NScD6l.png)

可以從左邊 Component Tree 的部分知道我用了哪一些的元件，要是需要更詳細的 .xml 文件內容就到文末的連結取用吧。

### 工作開始

首先我們先加入比較單純的部分，包括篩選清單以及刷新按鈕，我們篩選清單是使用一個 Spinner 結合 ArrayAdapter 來完成的。

> MainActivity.java
```
private ImageButton reloadImageButton;
private Spinner citySpinner;
private String[] cityArray;
private int selectedCity = -1;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    reloadImageButton = (ImageButton) findViewById(R.id.reload_imageButton);
    reloadImageButton.setOnClickListener(v -> dialogAndDownload());
    
    cityArray = getResources().getStringArray(R.array.city);
    
    ArrayAdapter<String> adapter = new ArrayAdapter<>(this, R.layout.my_spinner, cityArray);
    adapter.setDropDownViewResource(R.layout.my_spinner_dropdown);
    citySpinner = (Spinner) findViewById(R.id.spinner);
    citySpinner.setAdapter(adapter);
    citySpinner.setOnItemSelectedListener(spnOnItemSelected);
}
```

在這邊我們先約束了 ImageButton 以及 Spinner 的內容。這邊我有為了畫面的好看所以做了一點自定義的內容，像是 `R.layout.my_spinner_dropdown` 就是自己定義的一個 TextView 樣式，這類的內容一樣可以最後參考完整檔案的部分。

主要的概念就是給定一個 adapter 來約束它用哪一個 layout 顯示外觀，並且它的內容要顯示甚麼，接著在 setDropDownViewResource 如其名的就是在設置當 spinner 被點擊時下拉選單的樣式，最後再將這一個 adapter 指派給我們的 spinner 即可。

> MainActivity.java
```
private AdapterView.OnItemSelectedListener spnOnItemSelected = new AdapterView.OnItemSelectedListener() {
    @Override
    public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
        selectedCity = (int) id;
        dialogAndDownload();
    }

    @Override
    public void onNothingSelected(AdapterView<?> parent) {
    }
};

private void dialogAndDownload() {
    ProgressDialog dialog = new ProgressDialog(this, ProgressDialog.STYLE_SPINNER);
    dialog.setTitle("請稍後");
    dialog.setMessage("取得資料中...");
    dialog.setCancelable(false);
    dialog.show();
    dialog.dismiss();
}
```

在這邊的最後，我們也該把選單的觸碰以及 ImageButton 的觸碰行為也一併加上去。

到這邊其實我們也完成了快一半了，下一次的工作就是將下載下來的內容更新到我們一直還沒使用的 ListView 之中了喔。
