---
tags : Android,SharedPreferences
title : Android SharedPreferences
---
# SharedPreferences

- android.content.SharedPreferences類別
- 儲存簡單資料
  - key 與 value 的對應資料 (只限基本型態)
  ![](https://i.imgur.com/QGmsTUc.png)

   

  - ex: 帳號、設定、上一次登入時間、遊戲關卡或電子郵件
- 資料的儲存格式是XML檔


## 常用方法

### getSharedPreferences(String name, int mode)

#### 參數：檔案名稱String
定義此設定檔的檔名
不需要指定副檔名(ex: "atm"即寫入到atm.xml)

#### 參數：存取權限int
設定這個設定檔的存取權限
有四種選擇 (0,1,2,4)

| Value | Constant | 說明 |
| -------- | -------- | -------- |
| 0    | MODE_PRIVATE     | 只允許該 APP 存取>常用    |
| 1     | MODE_WORLD_READABLE	    | 所有 APP 都能讀取>不建議用     |
| 2    | MODE_WORLD_WRITEABLE     | 所有 APP 都能存取、寫入>不建議用      |
| 3     | MODE_MULTI_PROCESS    | 允許多個 process 同時存取>不建議用  |


### getPreferences(int mode)

- 建立一個讓目前的 Activity 使用的 SharedPreferences 檔案
  - 其他的 Activity 無法使用!
- 如果存在就將 SharedPreferences 指向該檔案；若不存在則會建立一個新的檔案，並指向該檔案
- 系統會在 /data/data/[package.name]/shared_prefs/ 目錄底下建立一個 Activity 名稱的 XML 檔，

## 寫入資料
- edit()：取得 Editor 物件
- SharedPreferences.Editor
  - commit()：直接將修改的結果寫入檔案
  - apply()：修改記憶體中的暫存資料，並以非同步式寫入檔案
  - put基本資料型態(key, value)：boolean, float, int, long, String, Set<String>
  - remove(key)
  - clear()

```shell=
Set<String> set = new HashSet<String>();
set.add("春天"); 
set.add("夏天");

SharedPreferences pref = getSharedPreferences("example", MODE_PRIVATE);
pref.edit()
    .putString("USER", "Boyd")
    .putInt("AGE", 24)
    .putFloat("WEIGHT", 55)
    .putStringSet("SEASON", set);
    .commit();
```



## 讀取

- get基本資料型態(key, defValue)：boolean, float, int, long, String, Set<String>
- contains(key)

```shell=
String userid = getSharedPreferences("example", MODE_PRIVATE)
            .getString("USER", "");
```


:::info
Ref:
- [使用SharedPreferences存取設定資料](https://litotom.com/ch7-1-sharedpreferences/)
- [如何使用 SharedPreferences 儲存簡易資料：寫入 與 讀取](https://spicyboyd.blogspot.com/2018/04/appandroid-sharedpreferences.html)
- [Day 27：SharedPreferences 資料存取](https://ithelp.ithome.com.tw/articles/10227342)
:::