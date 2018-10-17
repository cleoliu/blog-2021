---
layout: post
title: "[Network] HTTP Status Code"
categories: [Network, HTTP]
tags: [觀念]
description: 就是使用者（Client 端）發送一個 Request（請求），接著伺服器（Server 端）處理請求，並回傳一個 Response（回應）...
---


## HTTP狀態碼（HTTP Status Code）[1](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)
> 就是使用者（Client 端）發送一個 Request（請求），接著伺服器（Server 端）處理請求，並回傳一個 Response（回應）。
> 用以表示網頁伺服器超文字傳輸協定回應狀態的3位數位代碼。

<br/><br/>

***

## 1xx Informational 訊息
> 代表請求 **已被接受**，需要繼續處理。這類回應是 **臨時** 回應，只包含狀態行和某些可選的回應頭資訊，並以空行結束。意味著 **Server 期望收到更多請求 Payload**。

<br/><br/>

***

## 2xx Successful 成功
> 代表請求 **已成功被伺服器接收**、理解、並接受。

- **200 OK**
    請求已成功，請求所希望的回應頭或資料體將隨此回應返回。實際的回應將取決於所使用的請求方法。
    在GET請求中，回應將包含與請求的資源相對應的實體。
    在POST請求中，回應將包含描述或操作結果的實體。

<br/><br/>

***

## 3xx Redirection 重新導向
> 這類狀態碼代表需要用戶端採取進一步的操作才能完成請求。通常這些狀態碼用來重新導向，後續的請求位址（重新導向目標）。

- **301 Moved Permanently**
    被請求的資源已永久**移動到新位置**，這個回應也是可快取記憶體的。
    （像是 kimo 轉址到️ yahoo）
    ```
    HTTP/1.1 301 Moved Permanently
    Date: Wed, 15 Nov 1995 06:25:24 GMT
    Location: https://www.facebook.com/
    ```

- **302 Found**
    使用 get 和 post，這樣的**重新導向是臨時的**，你訪問的目標沒有符合當前預期，你必須先到別的地方再過來。
    （像是在點結帳時會先導向登入頁在導回結帳）

- **304 Not Modified**
    表示資源未被修改，因為請求頭指定的版本 If-Modified-Since 或 If-None-Match。在這種情況下，由於用戶端仍然**具有以前下載的副本**，因此不需要重新傳輸資源。

- **307 Temporary Redirect** 
    非永久轉移，狀況跟 302 一樣只是專門給提供給非 get 和 post 的 method 使用。

<br/><br/>

***

## 4xx Client Error 用戶端錯誤
> 代表了用戶端看起來可能發生了錯誤，妨礙了伺服器的處理。

- **400 Bad Request**
    由於明顯的用戶端錯誤，伺服器不能或不會處理該請求。
    （例如，**格式錯誤** 的請求語法，太大的大小，無效的請求訊息或欺騙性路由請求）

- **401 Unauthorized**
    可以重複提交一個包含恰當的Authorization頭資訊的請求。
    1. 當前請求需要 **用戶驗證** 或 **Token**。
    2. 如果當前請求已經包含了Authorization憑證，那麼401回應代表著伺服器驗證已經 **拒絕了那些憑證**。
    3. **禁止IP位址**時，有些網站狀態碼顯示的401，表示該特定位址被拒絕存取網站。

- **403 Forbidden**
    伺服器拒絕請求，可能是**用戶沒有足夠的權限**獲取資源。

- **404 Not Found**
    請求失敗，請求所希望得到的**資源未被在伺服器上發現**，或 Web Server 配置錯誤。
    舊資源因為某些內部的配置機制問題，已經永久的不可用，而且沒有任何可以跳轉的位址。
    （你要找的東西這裡沒有，回去確認看看吧 ）

- **405 Method Not Allowed**
    **網頁伺服器不允許的請求方法**（POST，GET，PUT，DELETE）。

- **408 Request Timeout**
    **請求超時**，當 server 想要關閉連線的時候就會丟這個訊息出來，告訴 Client 你的請求時間超過允許的時間我現在要關閉連線了。 

- **413 Payload Too Large**
    **傳輸的內容太大**，當 Client 收到這樣的錯誤通常都是在上傳檔案的時候，如果超過 web server 的限制設定，它就會回覆說你上傳的資料超過限制。

- **414 URI Too Long**
    **訪問的 URL 長度太長**，這個錯誤最常發生在使用 get 這個傳遞方式。

- **415 Unsupported Media Type**
    訪問 **header 所指定的 Media Type 不支援**，在 header 裡面可以指定 request 的存取資料類型給 server 收到時間查，這是比較嚴謹的存取限制，因此如果 client 要存取一個 jpg 的圖片，但是 server上只有支援 gif，這樣的請求明顯不符合要求就會收到這樣的回應。 

<br/><br/>

***

## 5xx Server Error 伺服器錯誤
> 伺服器在處理請求的過程中有錯誤或者異常狀態發生，也有可能是伺服器意識到以當前的軟硬體資源無法完成對請求的處理。

- **500 Internal Server Error**
    Client 訪問的位置沒問題，但是 Server 接收到請求後開始處理時程式發生異常狀況，簡單來說就是**程式執行時掛掉了**，無法返回正常的內容時就會用這個訊息通知，通常遇到這個錯誤 Server 上可以看到程式發生 Crash 。

- **502 Bad Gateway**
    Gateway 匝道器（像load balance或防火牆會決定路由的網路設備），他會幫你轉送你的請求和回傳封包。
    那在網路上兩點間的傳輸會有所謂的 timeout，這個是避免單一個點會傻傻的等待，所以當時間一到他就必須做個決定回傳結果從server 到網路設備上的關卡只要有一關遇到問題就會出現 502 
    (502內容包含504) [1](https://notfalse.net/50/http-intermediary) 匝道

    <img src="https://s3.amazonaws.com/notejoy/note_images/99942.1.Image%202018-08-24%20at%20%E4%B8%8A%E5%8D%8811.46.22.png" width="60%" height="30%" />
    
    <img src="https://s3.amazonaws.com/notejoy/note_images/99942.1.2018-10-17%20%E4%B8%8A%E5%8D%88%2010-37-44.jpg" width="60%" height="30%" />

- **503 Service Unavailable**
    由於**臨時的伺服器維護**或者過載，伺服器當前無法處理請求。這個狀況是暫時的，並且將在一段時間以後恢復。
    （相當於準備開店）

- **504 Gateway Timeout**
    伺服器上的**服務沒有回應**。(**預設閘道愈時**)
    作為閘道器或者代理工作的伺服器嘗試執行請求時，未能及時從上游伺服器（URI標識出的伺服器，例如HTTP、FTP、LDAP）或者輔助伺服器（例如DNS）收到回應。
    網路設備的延遲會讓client和server雙方的有效時間對不起來，避免server或client收到內容時都已經過了很久，網路設備延遲到了一定程度就會送這個訊息。 

    <img src="https://s3.amazonaws.com/notejoy/note_images/99942.1.Image%202018-08-24%20at%20%E4%B8%8A%E5%8D%8811.11.23.png" width="70%" height="30%" />
    

​<br/><br/>

***

## ref：
- [https://notfalse.net/48/http-status-codes](https://notfalse.net/48/http-status-codes)

