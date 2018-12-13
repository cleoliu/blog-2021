---
layout: post
title: "[Python ChatRobot] 01. Wechat Robot-itchat"
categories: [Python]
tags: [Python, itchat, Robot]
description: 實現微信個人號聊天機器人通過自定義消息處理方法加入聊天功能...
---

> itchat 可以實現一個聊天機器人的 server，可以主動發送訊息及文件，也可以做為回覆機器人，登入方式是透過 wechat 網頁版，掃 QRcode 登入（現在為了避免濫用，wechat 官方網頁版有限制新用戶不可使用，所以請準備一個有在使用的 wechat 帳號），在啟動之後 server 會持續接聽，在後面接的功能（如 api）失敗了也不會中斷 server，可以串接的功能像是 圖靈回覆、翻譯 api、自動報時、爬蟲程式等等，這篇就先說明 itchat 的基本用法

<br>

### 環境
- Python 3.5.1

### 安裝
```bash
$ pip install itchat
```

<br>

***

## 入門實例 [1](https://itchat.readthedocs.io/zh/latest/)
- 發一條信息：robot `主動` 發送訊息

    ```python
    import itchat

    itchat.auto_login()
    itchat.send('Hello, filehelper', toUserName='filehelper')
    ```

- 回覆訊息：robot `被動`，收到對方傳來的訊息才做回覆

    ```python
    import itchat

    @itchat.msg_register(itchat.content.TEXT) #回覆的裝飾器
    def text_reply(msg):
        return msg.text

    itchat.auto_login()
    itchat.run()
    ```

<br>

***

## 登入/登出 [1](https://itchat.readthedocs.io/zh/latest/intro/login/)

- **登入**：在運行下面程式後會彈出 QRcode 的圖片檔，需掃碼登入

    ```python
    itchat.auto_login()
    ```

- 如果像是要運行在 linux Server，可用命令行顯示 **QRcode**

    ```python
    itchat.auto_login(enableCmdQR=True)
    # 如部分的linux系統，塊字符的寬度為一個字符（正常應為兩字符），故賦值為2
    itchat.auto_login(enableCmdQR=2) 
    # 默認控制台背景色為暗色（黑色），若背景色為淺色（白色），可以將enableCmdQR賦值為負值
    itchat.auto_login(enableCmdQR=-1)
    ```

- 退出程序後 **暫存登陸狀態**：通過如下命令登陸，即使程序關閉，一定時間內重新開啟也可以不用重新掃碼，該方法會生成一個靜態文件 _itchat.pkl_，用於存儲登陸的狀態。
    
    ```python
    itchat.auto_login(hotReload=True)
    ```

- **登出**

    ```python
    itchat.logout()
    ```

***

## 發送方法 [1](https://itchat.readthedocs.io/zh/latest/intro/reply/)

> 有多種 function 可發送不同類型的消息

<br>

- `send`：大部分類型都可以用這個方法

    ```python
    # 發送訊息
    itchat.send(msg='Text Message', toUserName=None) #toUserName為空則發送給自己

    # 發送文件
    itchat.send('@img@%s' % 'gz.gif')
    itchat.send('@fil@%s' % 'xlsx.xlsx')
    itchat.send('@vid@%s' % 'demo.mp4')
    ```

- `send_msg`

    ```python
    itchat.send_msg(msg='Text Message', toUserName=None)
    itchat.send_msg('Hello world')
    ```

- `send_file`

    ```python
    itchat.send_file(fileDir, toUserName=None)
    itchat.send_file('xlsx.xlsx')
    ```

- `send_img`

    ```python
    itchat.send_img(fileDir, toUserName=None)
    itchat.send_img('gz.gif')
    ```

- `send_video`

    ```python
    itchat.send_video(fileDir, toUserName=None)
    itchat.send_file('demo.mp4')
    ```

***

## 通訊錄 [1](https://itchat.readthedocs.io/zh/latest/intro/contact/)

> 微信有三種帳號類型，分別為：好友、公眾號、群聊

<br>

### 用戶好友

- 使用 `search_friends` 方法可以搜尋用戶

    ```python
    # 獲取自己的用戶信息，返回自己的屬性字典
    itchat.search_friends()
    # 獲取特定UserName的用戶信息
    itchat.search_friends(userName='@abcdefg1234567')
    # 獲取備註、微信號、暱稱任何一項等於name鍵值的用戶
    itchat.search_friends(name='littlecodersh')
    # 獲取備註、微信號、暱稱分別等於鍵值的用戶
    itchat.search_friends(wechatAccount='littlecodersh')
    # 三、四項功能可以一同使用
    itchat.search_friends(name='LittleCoder机器人', wechatAccount='littlecodersh')
    ```

- 更新用戶信息

    ```python
    memberList = itchat.update_friend('@abcdefg1234567')
    ```

<br>

### 公眾號

```python
# 獲取特定UserName的公眾號，返回值為一個字典
itchat.search_mps(userName='@abcdefg1234567')
# 獲取名字中含有特定字符的公眾號，返回值為一個字典的列表
itchat.search_mps(name='LittleCoder')
# 以下方法相當於僅特定了UserName
itchat.search_mps(userName='@abcdefg1234567', name='LittleCoder')
```

<br>

### 群聊

- **群聊列表**

    ```python
    # 取特定UserName的群聊，返回值為一個字典
    itchat.search_chatrooms(userName='@@abcdefg1234567')
    # 獲取名字中含有特定字符的群聊，返回值為一個字典的列表
    itchat.search_chatrooms(name='LittleCoder')
    # 以下方法相當於僅特定了UserName
    itchat.search_chatrooms(userName='@@abcdefg1234567', name='LittleCoder')
    ```

- 獲取 **群聊用戶列表**
    - 群聊在首次獲取中不會獲取群聊的用戶列表，所以需要調用該命令才能獲取群聊的成員
    - 鍵為 True 將可以更新群聊列表並返回通訊錄中保存的群聊列表

    ```python
    memberList = itchat.update_chatroom('@@abcdefg1234567', detailedMember=True)
    ```

- **創建群聊**
    - 超過 40 人的群聊無法使用直接加入的加入方式
    - 刪除群聊需要本賬號為群管理員

    ```python
    memberList = itchat.get_friends()[1:]
    # 創建群聊，topic鍵值為群聊名
    chatroomUserName = itchat.create_chatroom(memberList, 'test chatroom')
    # 删除群聊内的用户
    itchat.delete_member_from_chatroom(chatroomUserName, memberList[0])
    # 增加用户进入群聊,將用戶加入群聊有直接加入與發送邀請，通過useInvitation設置
    itchat.add_member_into_chatroom(chatroomUserName, memberList[0], useInvitation=False) 
    ```

<br><br>

***

## ref

- [itchat](https://itchat.readthedocs.io/zh/latest/)

<br><br>