---
tags : Android,HTTP
title : Android HTTP
---
# HTTP

## HTTP協議的特點
* 支援客戶／伺服器模式
* 簡單快速：客戶向服務端請求服務時，只需傳送請求方式和路徑。
* 靈活：允許傳輸任意型別的資料物件(由Content-Type加以標記)
* 無連線：每次回應一個請求，完成以後就斷開連線。
* 無狀態：伺服器不儲存瀏覽器的任何資訊。每次提交的請求之間沒有關聯。

## HttpURLConnection
* 一種多用途、輕量極的HTTP客戶端
* 使用它來進行HTTP操作可以適用於大多數的應用程序
* 它提供的API的比較簡單，但也使我們可以更加容易地去使用和擴展。


### 使用HttpURLConnection的步驟：

1. 創建一個URL
2. 用URL的openConnection( )來獲取HttpURLConnection的例項
```shell= 
URL url = new URL("http://blog.csdn.net/checkiming");
HttpURLConnection connection = (HttpURLConnection)url.openConnection();
```
3. 設置HTTP請求使用的方法:GET、POST，或是其他(EX: PUT)

GET、POST方式的差別:
|  | GET | POST |
| -------- | -------- | -------- |
| 方式     | 從伺服器那裡獲取到資料 | 提交資料給伺服器 |
| 資料大小    | 大於2KB(URL長度限制)     | 沒有此限制    |
| 安全性 | 適用非敏感資料(參數會顯示在地址欄上) | 較安全 |

```shell= 
connection.setRequestMethod("GET")
//connection.setRequestMethod("POST")
```
4. 設置連接超時，讀取超時的毫秒數，以及服務器希望得到的一些消息
(這部分可以根據自己的需求情況寫的)
```shell= 
connection.setConnectTimeout(6000);
connection.setReadTimeout(6000);
```
5. 用getInputStream()方法獲得伺服器返回的輸入流，然後進行讀取
```shell= 
InputStream in = connection.getInputStream();
```
6. 用disconnect()方法將HTTP連接關掉
```shell= 
connection.disconnect();
```
:::info
Ref:
- [Android的HTTP請求方式](https://www.itread01.com/content/1546129107.html)
- [Android HTTP请求方式:HttpURLConnection](https://www.runoob.com/w3cnote/android-tutorial-httpurlconnection.html)
- [Android HttpURLConnection 之POST&GET手把手完整教學解析](https://www.itread01.com/content/1546129107.html)
- [Android 如何使用 GET & POST 取得資料](http://blog.tonycube.com/2011/11/androidget-post.html)
:::