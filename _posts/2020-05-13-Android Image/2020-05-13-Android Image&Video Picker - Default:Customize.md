---
layout: post
title: Android的圖片/影片選擇器(瀏覽器) - Default/Customize
tags: [Android, Java]
---

## 前言
在現在的手機使用情境中，有越來越多的機會讓使用者去運用他們的圖片或影片內容，例如現在絕大多數的社群軟體都會需要用到圖片/影片的上傳，而這相對應的就會需要一個方便且適合的檔案選取方式，或許是單一選取、複數選取等等，也可能會因為需要根據自己的 UI 設計而有所調整，所以如何將圖片/影片選擇器進行客製化也是個重要的應用。

在這篇文章中，我會先介紹最基本 Default 選擇器的使用方式，要是沒有特殊需求使用預設的選擇器也不失是一個簡單且方便的方式。

另外也會介紹一個類似現在大多社群軟體(e.g.: Line, FB, IG)，所使用的網格式清單選擇器，並且同時可以選擇圖片和影片。

---

### 系統預設選擇器

在實際開始運作我們的選擇器功能前，我們必須去詢問使用者是否願意提供我們存取的權限，這邊我們先在 `AndroidManifest.xml` 中加入權限需求。

``` xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <application
        ...
    </application>
</manifest>
```

然後我們回到 MainActivity.java 中，在這邊我們可以相應的在 activity_main.xml 裡相對做畫面調整就不贅述，首先我們先加入一個權限檢查的動作，來確認是否正確取得需要的權限，若是沒有的話我們也可以選擇關閉 APP或是再次索取等等的動作。

``` java
if (ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
    // 未取得權限，向使用者索取
    ActivityCompat.requestPermissions(this, PERMISSIONS_STORAGE, REQUEST_EXTERNAL_STORAGE);
} else {
    // 以取得權限，做出相應反應亦可不做反應
}
```

接著，我們再加入一個 button 來進行我們選擇器的觸發，並且在 button 的 OnClickListener 中加入我們打算進行的預設圖片選擇器跳轉功能。

``` java
public class MainActivity extends AppCompatActivity {
    Button defaultButton;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        defaultButton = (Button) findViewById(R.id.default_btn);
        ...

        defaultButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                intent.setType("image/*");
                // 若要選取影片則改為 intent.setType("video/*");
                // 設置選取的檔案的 MIME 屬性，要是想過濾特定的格式也可以把 * 換成目標檔案格式
                intent.putExtra(Intent.EXTRA_ALLOW_MULTIPLE, true);
                // 設置可以多選的選取屬性，若不需要多選可以移除
                intent.setAction(Intent.ACTION_GET_CONTENT);
                // 設置動作類型，這邊只進行讀取
                startActivityForResult(Intent.createChooser(intent,"Select Picture"), PICK_IMAGE_MULTIPLE);
                // 交辦 intent 以及接收回傳選取結果
            }
        });
    }
}
```

基本上到這邊，其實就已經可以做到開啟預設選擇器的動作了，不過今天既然稱為選擇器，那就是要做選取的動作，因此如何處理回傳回來的結果也是一個重點，在上面我們是以 startActivityForResult() 啟動的，因此在處理回傳的地方我們也就會是使用 onActivityResult 來處理回傳。

``` java
@Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        try {
            if (requestCode == PICK_IMAGE_MULTIPLE && resultCode == RESULT_OK && null != data) {
                Toast.makeText(this, "Success to pick image", Toast.LENGTH_LONG).show();
                // 這邊便可以對輸入的 data 進行我們想要做的處理
            }
        } catch (Exception e) {
            Toast.makeText(this, "Something wrong", Toast.LENGTH_LONG).show();
        }
        super.onActivityResult(requestCode, resultCode, data);
    }
```

以上便是針對預設選擇器部分的使用方式，既然是預設那就是最基礎的方式，所以他的限制也較多，自由度沒有那麼高，因此我們接下來就來試著做一個自己的選擇器吧。

---

### 自訂義圖片/影片選擇器

在這個方法中，我們會用到的幾個比較重要重點有：
1. 因為我們在讀檔的顯示有可能會是大量的內容，因此在這邊我選擇使用 RecyclerView ，並搭配 GridLayoutManager 來進行管理和顯示。
2. 因為我們不用預設的選擇器，因此讀取檔案的工作我們必須自己處理。

---

OK，在簡單介紹完我們接下來要做的工作後，我們就開始吧！
首先我們先將檔案讀取的方式做好，後面就可以根據我們要的資料來做顯示。

在讀取多媒體檔案的這個部分，我採用 MediaStore 這個 Android 提供的方法來進行，並且結合 CursorLoader 來將我們資料進行載入。

其實在這個過程的動作有點像是在進行資料庫的讀取，所以我們先上程式碼來看一下。

``` java
private void getLocalMediaUri() {
        Uri queryUri = MediaStore.Files.getContentUri("external");

        String[] projection = {
                MediaStore.Files.FileColumns._ID,
                MediaStore.Files.FileColumns.DATA,
                MediaStore.Files.FileColumns.DATE_ADDED,
                MediaStore.Files.FileColumns.MEDIA_TYPE,
                MediaStore.Files.FileColumns.MIME_TYPE,
                "duration",
                MediaStore.Files.FileColumns.TITLE
        };

        String selection = MediaStore.Files.FileColumns.MEDIA_TYPE + "="
                + MediaStore.Files.FileColumns.MEDIA_TYPE_IMAGE + " OR "
                + MediaStore.Files.FileColumns.MEDIA_TYPE + "="
                + MediaStore.Files.FileColumns.MEDIA_TYPE_VIDEO;

        String order = MediaStore.Files.FileColumns.DATE_ADDED + " DESC";

        CursorLoader cursorLoader = new CursorLoader(this, queryUri, projection, selection, null, order);
        Cursor cursor = cursorLoader.loadInBackground();

        // MEDIA_TYPE: IMAGE = 1; VIDEO = 3;
        int indexFileTitle = cursor.getColumnIndexOrThrow(MediaStore.Files.FileColumns.TITLE);
        int indexFilePath = cursor.getColumnIndexOrThrow(MediaStore.Files.FileColumns.DATA);
        int indexFileTpye = cursor.getColumnIndexOrThrow(MediaStore.Files.FileColumns.MEDIA_TYPE);
        int indexFileMime = cursor.getColumnIndexOrThrow(MediaStore.Files.FileColumns.MIME_TYPE);
        int indexVideoDuration = cursor.getColumnIndexOrThrow(MediaStore.Files.FileColumns.DURATION);

        while (cursor.moveToNext()) {
            inputData.add(cursor.getString(indexFilePath));

            String title = cursor.getString(indexFileTitle);
            String uri = cursor.getString(indexFilePath);
            String type = cursor.getString(indexFileTpye);
            String mime = cursor.getString(indexFileMime);
            Integer duration = cursor.getInt(indexVideoDuration);
            SelectMedia file;
            if (type.equals("1")) {
                file = new SelectMedia(title, SelectMedia.FileType.IMAGE, uri, mime, null, null);
            } else {
                file = new SelectMedia(title, SelectMedia.FileType.VIDEO, uri, mime, null, duration);
            }
            inputMediaArrayList.add(file);
        }
        cursor.close();
    }
```

這編最重要的部分是 `CursorLoader` 的使用，所以我們先看一下他的原始碼：

``` java
/**
     * Creates a fully-specified CursorLoader.  See
     * {@link ContentResolver#query(Uri, String[], String, String[], String)
     * ContentResolver.query()} for documentation on the meaning of the
     * parameters.  These will be passed as-is to that call.
     */
    public CursorLoader(@NonNull Context context, @NonNull Uri uri, @Nullable String[] projection,
            @Nullable String selection, @Nullable String[] selectionArgs,
            @Nullable String sortOrder) {
        super(context);
        mObserver = new ForceLoadContentObserver();
        mUri = uri;
        mProjection = projection;
        mSelection = selection;
        mSelectionArgs = selectionArgs;
        mSortOrder = sortOrder;
    }
```

我們在這裡面可以看到他其實可以塞入的變數有 6 個，`context、uri、projection、selection、selectionArgs、sortOrder`，除了context之外他們分別代表著：
* uri - 要擷取內容的 URI。
* projection - 要傳回的欄清單。傳送 null 將會傳回無效的所有欄。
* selection - 篩選器會採用 SQL WHERE 子句的格式 (WHERE 本身除外) 宣告要傳回的列。 傳送 null 將會傳回指定 URI 的所有列。
* selectionArgs - 您可能會在選取項目中包含 ?s，而其會由來自 selectionArgs 的值按照其出現在選取項目中的順序所取代。 值將會繫結為字串。
* sortOrder - 如何採用 SQL ORDER BY 子句的格式 (ORDER BY 本身除外) 來排列各列的順序。 傳遞 null 將會使用預設的排序順序，其可能是無排序順序。

了解完這一個部分之後，我們再回頭看我們的程式碼，也根據這幾個欄位分別建立了相對的變數，由於這邊我希望影片跟圖片同時讀取一並顯示在我們自定義的畫面中，所以可以看到我使用的 constant `MediaStore.Files.FileColumns.` 下面的 constant，要是單純想使用圖片、影片、或是音訊檔案的話也可以考慮使用 `MediaStore.Video.VideoColumns.` 或是 `MediaStore.Images.ImageColumns.` 這類的 constant 他們可以更有針對性的去使用資源。另外這些 constant 下面都還有許多屬性可以依據你需要的功能或是資訊相對應的去取用，這邊就不贅述。

接著在指派完我們的 CursorLoader 後便是開始進行我們把資訊讀出後的處理工作，像是對我們需要的資訊做好宣告並藉由 while 移動 CursorLoader 的指標來將我們讀出的所有檔案訊息加以處理，在這裡比較需要注意的是我有宣告一個自己的類別 SelectMedia 讓我我後面在使用的時後可以更方便地做存取。

其實到了這邊我們所有需要做的工作基本已經完成，接下來就只是將接出來的檔案資訊根據畫面需求依序放入畫面元件中就可以了。

最後附上我依照 line 的圖片影片選擇器刻出來的畫面：
![畫面截圖](https://ithelp.ithome.com.tw/upload/images/20200513/20125739vWXMz24qGy.png)

不過在這邊我先補充說一下在把圖檔塞進自定義選擇畫面時一定要去注意圖片的大小，因為現在相機拍出的圖片每一張都動輒1MB、2MB甚至更大的，所以一定要做縮圖的動作才不會讓資源一次使用過多而城市被強制關閉。

另外就是因為要進行縮圖的處理，所以相對應的縮圖製作時機、recycleview 在資源釋放與重用時的管理也都是可以好好研究的問題，這邊的使用我也只是使用了一個我認為比較不影響使用體驗的方式，但我相信還會有更好的資源重用方法或是縮圖管理的方式可以提升使用者體驗。這部分就留給大家自己研究~未來有機會我再填上這個坑！
