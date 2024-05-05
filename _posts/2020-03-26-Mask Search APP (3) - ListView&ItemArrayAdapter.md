---
layout: post
title: 自己做自己的口罩查詢APP (3) - 通通變成我的形狀吧 ListView&ItemArrayAdapter 
tags: [Android, Java]
---

## 前言
其實到了這一篇基本上就已經到了最後，我們再來只要將切好的資料放進對應好的欄位裡面就完成了

那這邊為了將盡量做到 MVC 的架構(對 Android 還是不夠熟沒辦法做到很完整的切割...)，所以在這邊我將 ListView 中的 item 取出個別處理，這樣的好處還有我們可以針對 ListView 中每個欄位的顯示樣式做自定義，讓顯示的內容更符合我們的需求。

## 開工
這一篇簡單的概念有了，就讓我們開始吧~

首先我們得加入一個 list_item.xml，他將會是我們後面顯示的欄位
![https://ithelp.ithome.com.tw/upload/images/20200331/20125739tm5mCuwneS.png](https://ithelp.ithome.com.tw/upload/images/20200331/20125739tm5mCuwneS.png)

在這邊一樣先附上在這邊一樣先附上截圖，可以自己刻自己需要的樣式不一定要一模一樣，若是不清楚可以在最後的連結去取得檔案來看。

接著我們就要來撰寫這些 item 的運作了，首先我們需要新增一個 ItemArrayAdapter.java 它的功用會是我們後續將處理完的資料放進來，再將其對應到相對欄位的相對元件之中的控制。

以及未來我們 ListView 內容在初始化以及刷新時的控制機制也會在這邊進行約束。

> ItemArrayAdapter.java
```
@Override
public int getCount() {
    return this.maskInfoList.size();
}

@Override
public String[] getItem(int index) {
    return this.maskInfoList.get(index);
}

@Override
public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
    View row = convertView;
    ItemViewHolder viewHolder;
    if (row == null) {
        LayoutInflater inflater = (LayoutInflater) this.getContext().getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        row = inflater.inflate(R.layout.list_item, parent, false);
        viewHolder = new ItemViewHolder();
        viewHolder.pharmacyname = (TextView) row.findViewById(R.id.pharmacy_name_textView);
        viewHolder.pharmacyaddress = (TextView) row.findViewById(R.id.pharmacy_address_textView);
        viewHolder.telephone = (TextView) row.findViewById(R.id.telephone_textView);
        viewHolder.adultmaskamount = (TextView) row.findViewById(R.id.adult_amount_textView);
        viewHolder.childmaskamount = (TextView) row.findViewById(R.id.child_amount_textView);
        viewHolder.updatetime = (TextView) row.findViewById(R.id.update_time_textView);
        row.setTag(viewHolder);
    } else {
        viewHolder = (ItemViewHolder) row.getTag();
    }

    String[] stat = getItem(position);
    viewHolder.pharmacyname.setText(stat[1]);
    viewHolder.pharmacyaddress.setText(stat[2]);
    viewHolder.telephone.setText(stat[3]);
    viewHolder.adultmaskamount.setText("成人口罩剩餘數: " + stat[4]);
    viewHolder.childmaskamount.setText("兒童口罩剩餘數: " + stat[5]);
    viewHolder.updatetime.setText(stat[6]);

    Log.v("Test", "setvalue");
    return row;
}
```

這一個部分先是針對數量、內容、以及將這些內容放到對應來未的機制進行規範。

接著就是要來針對初始化以及刷新的部分進行撰寫，第一個 PersonAdapter() 就是我們初始化的部分，refresh() 如其名的就是刷新的部分，再刷新的部分我們使用notifyDataSetChanged() 來處理資料刷新，使用這個方法的話就只會針對表格內有異動的欄位進行修改，而不需要將整個表格物件重置再建立，雖然在目前內容不吃資源的情況下差別不大，但要是我們今天會讀取較大資料或是圖檔時就會造成大量重複刷新的資源耗損。

```
public void PersonAdapter(ArrayList<String[]> list) {
    maskInfoList = list;
}

public void refresh(ArrayList<String[]> list) {
    maskInfoList = list;
    notifyDataSetChanged();
}
```

-----

在完成了 ItemArrayAdapter.java 之後，我們就要把我們切丸的資料正式放入我們的 ListView 之中了，我們回到 MainActivity.java 中。

我們需要對  進行一些修改，讓這部分可以在下載資料的前後做一些處理，來讓資料順利放入。

```
private void dialogAndDownload() {
    ProgressDialog dialog = new ProgressDialog(this, ProgressDialog.STYLE_SPINNER);
    dialog.setTitle("請稍後");
    dialog.setMessage("取得資料中...");
    dialog.setCancelable(false);
    dialog.show();

    if (isFirst) {
        initialListAndAdapter();
    } else {
        maskInfoList.clear();
        itemArrayAdapter.clear();
    }

    new Thread(() -> {
        downloadData();
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                if (isFirst) {
                    isFirst = false;
                    itemArrayAdapter.PersonAdapter((ArrayList<String[]>) maskInfoList);
                } else {
                    itemArrayAdapter.refresh((ArrayList<String[]>) maskInfoList);
                }
            }
        });
        dialog.dismiss();
    }).start();
}
```

接著補上初始化我們 listView 以及 itemArrayAdapter 的 function。

```
private void initialListAndAdapter() {
    maskInfoList.clear();
    listView = (ListView) findViewById(R.id.maskListView);
    itemArrayAdapter = new ItemArrayAdapter(getApplicationContext(), R.layout.list_item);

    Parcelable state = listView.onSaveInstanceState();
    listView.setAdapter(itemArrayAdapter);
    listView.onRestoreInstanceState(state);
}
```

最後在字串處理完後的地方加入 `maskInfoList.add(row);` 讓處理好的資料夾入 ListView 會用到的 List 之中。

```
while ((csvLine = reader.readLine()) != null) {
    String[] row = csvLine.split(",");
    if (selectedCity >= 0 && row[2].substring(0, 3).equals(cityArray[selectedCity])) {
        maskInfoList.add(row);
    }
}
```

到這邊基本上應該只要補上一些殘餘的宣告就完成我們所有的工作了。
這一篇盡量省略一些比較累贅的部分，因此若是只照著上面的東西貼上去一定沒辦法直接跑，會需要加一些宣告跟小東西，不過這部分就不贅述，不然實在有點多餘。


