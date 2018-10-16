---
layout: post
title: "[MVC教學] Route Controller"
categories: [ASP•NET, C＃]
tags: [實做]
description: 像是郵差依據包裹上的地址，透過地圖找到目的地然後投遞。...
---


## 什麼是Route？ 
- 像是郵差依據包裹上的地址，透過地圖找到目的地然後投遞。
- 而對應到程式中 [網址] 就是你包裹要送達的目的地址，Route 就是 [地圖]，而網址的 [參數] 就是你要投遞的包裹。

<br/><br/>

***

## 不須變數的Route
- 不須變化的靜態網頁，例如網站關於我介紹。
```
    http://localhost/Home/Index
```

<br/><br/>

1. 執行後在首頁點擊 [關於] 
2. 關於頁的網址為 /Home/About
3. 再點擊一次網頁的 [關於] 按鈕
4. 顯示 "Your application description page."

<br/><br/>

### HomeController
C:\Users\USER\Documents\GIT\Keep_accounts\mymoney\mymoney\Controllers\HomeController.cs
![](https://s3.amazonaws.com/notejoy/note_images/146088.2.2018-10-16%20%E4%B8%8A%E5%8D%88%2011-12-18.jpg)
​
### RouteConfig
C:\Users\USER\Documents\GIT\Keep_accounts\mymoney\mymoney\App_Start\RouteConfig.cd
![](https://s3.amazonaws.com/notejoy/note_images/146088.1.2018-10-16%20%E4%B8%8A%E5%8D%88%2011-14-12.jpg)
​
- **Name**︰你對於這個Route的命名
- **Url**︰網址條件，當網址符合這個條件特徵時，就會依據這個Route的指示去找對應執行的程式碼
- **defaults**︰
    - 參數的預設值
    - id = UrlParameter.Optional表示它是選擇性的，如果沒有沒關係
- 舉個栗子︰
```
    Route Url︰ /{controller}/{action}/{id}  
    關於頁網址︰ /Home/About                #id現在沒有值
    首頁網址︰   /                          #啥都沒有 --->參考Default=/Home/Index 
```

<br/><br/>

***

## 建個新 Controller 在繼續
- 在 Controller 資料夾按下滑鼠右鍵 -> 加入 -> 控制器
- 輸入 [RouteTestController] 後按下 [加入]
- 選擇 [MVC 5 控制器 - 空白] 後按下 [新增]

<br/><br/>

***

## 單變數與多變數的Route

### 單變數
- 單純的 CRUD（Create、Read、Update、Delete）操作，基本上只需要 index 就已足夠
```
    http://localhost/Home/Index/1
```

### 多變數
- 比較複雜的操作，例如搜尋分頁功能時，需要同時傳送 第幾頁 與 搜尋條件 ，如此就需要同時傳送多個變數至 Action
```
    http://localhost/Home/Index/?id=1&page=3
```

<br/><br/>

### RouteTestController

C:\Users\USER\Documents\GIT\Keep_accounts\mymoney\mymoney\Controllers\RouteTestController.cs
```csharp
namespace mymoney.Controllers
{
  public class RouteTestController: Controller
  {
    public ActionResult Index() //不輸入任何參數
    {
      return Content("這是index");
    }

    public ActionResult Index2(string id) //單變數
    {
      return Content(
        String.Format("Id值為{0}", id);
        );
    }
                
    public ActionResult Index3(string id, string page) //多變數
    {
      return Content(
        String.Format("Id值為{0}, page值為{1}", id, page);
        );
    }
  }
}
```

- 舉個栗子：
```
  Route Url : 　/{controller}/{action}/{id}  
  Index頁：　　　/RouteTest/Index
  代個id： 　　　/RouteTest/Index2/cleo
  代個?id：　　　/RouteTest/Index2/?id=cleo
  代個id和page：/RouteTest/Index3/id=cleo&page=911
```



<br/><br/>

***
### 參考文獻
- https://progressbar.tw/posts/105
- https://ithelp.ithome.com.tw/articles/10158071






