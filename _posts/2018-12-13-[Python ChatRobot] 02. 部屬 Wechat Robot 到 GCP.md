---
layout: post
title: "[Python ChatRobot] 02. 部屬 Wechat Robot 到 GCP"
categories: [ChatRobot]
tags: [Python, itchat, Robot, GCP]
featured-img: emile-perron-190221
---

- 首先先到 google 申請 GCP 服務吧，目前（2018）提供新加入成員一年 & 300 美元的試用額度。

    [https://console.cloud.google.com/](https://console.cloud.google.com/)

- 點選免費試用，會要求你填寫信用卡資訊，送出後會收到刷卡通知，之後會在收到退刷，時間到期或是金額用畢會先通知不會自動扣款。

<br>

- 申請成功後會自動導向到 GCP 的控制頁面。
- 點選左上角的服務選單 > 「Compute Engine」 > 「VM 執行個體」。

    ![](https://s3.amazonaws.com/notejoy/note_images/180971.1.2018-12-13%20%E4%B8%8A%E5%8D%88%2010-38-15.jpg)

- 點選上方的「建立執行個體」。

    ![](https://s3.amazonaws.com/notejoy/note_images/180971.1.2018-12-13%20%E4%B8%8A%E5%8D%88%2010-39-10.jpg)

- 開始進行虛擬機的設定。

    ![](https://s3.amazonaws.com/notejoy/note_images/180971.1.2018-12-13%20%E4%B8%8A%E5%8D%88%2010-41-36.jpg)

    1. 設定名稱
    2. 設定區域：如果想要減少連線延遲，可以選離台灣最近的是 asia-east1，不過區域價格有差，可以自行取捨了。
    3. 機器類型：使用「微型」或「小型」就可以了。
    4. 開機磁碟：點選「變更」更換，這邊是選擇「Ubuntu 16.04 LTS」。
    5. 防火牆：兩個都打勾。

<br>

- 點選「建立」後，需要一點時間等他建立，建立完成後，名稱前方會有綠色的勾勾。

    ![](https://s3.amazonaws.com/notejoy/note_images/180971.1.2018-12-13%20%E4%B8%8A%E5%8D%88%2010-50-05.jpg)

- 在連線的下拉選單可以選擇不同的連線方式，windows 沒有內建 SSH，所以可以點選「在瀏覽器視窗中開啟」，開啟的終端機就已經幫你連線到雲端 VM 了。

    ![](https://s3.amazonaws.com/notejoy/note_images/180971.1.2018-12-13%20%E4%B8%8A%E5%8D%88%2010-53-29.jpg)

<br>

***

## Install

### Install python2.7 Pip (on Ubuntu 16.04)

- Connect to SSH and Update your System Software

    ```bash
    $ sudo apt-get update && sudo apt-get -y upgrade
    ```

- Install Pip on Ubuntu 16.04

    ```bash
    $ sudo apt-get install python-pip
    $ pip -V
    >>>pip 8.1.1 from /usr/lib/python2.7/dist-packages (python 2.7)
    $ pip install --upgrade pip
    ```

- 升級 pip 後如果出現 ImportError: cannot import name main，可以做以下操作：

    ```bash
    $ cd /usr/bin
    $ sudo vim pip
    ```

    ```python
    # 把下面的三行
    from pip import main
    if __name__ == '__main__':
    sys.exit(main())

    # 換成下面的三行
    from pip import __main__
    if __name__ == '__main__':
    sys.exit(__main__._main())
    ```

<br>

## 安裝虛擬環境 

- virtualenv

    ```bash
    $ sudo pip install virtualenv
    $ virtualenv VENV #建立虛擬環境
    $ source VENV/bin/activate #進入虛擬環境
    ```

<br>

## 安裝 python 3.5

- 3.5

    ```bash
    (VENV) $ sudo apt-get update 
    (VENV) $ sudo apt-get install python3.5 
    (VENV) $ sudo cp /usr/bin/python /usr/bin/python_bf
    (VENV) $ whereis python3.5  #看安裝路徑
    (VENV) $ sudo rm /usr/bin/python #删除原有的Python连接文件
    (VENV) $ sudo ln -s /usr/bin/python3.5 /usr/bin/python #建立指向Python3.5的連結
    (VENV) $ PATH=/usr/bin:$PATH #環境變量
    (VENV) $ python3 -V 或 $ python --version
    ```

- python3 的 pip

    ```bash
    (VENV) $ curl https://bootstrap.pypa.io/get-pip.py | sudo python3.5
    (VENV) $ pip3 -V
    ```

<br>

## 安裝 itchat

- itchat

    ```bash
    (VENV) $ sudo pip install itchat
    (VENV) $ sudo pip install pillow
    (VENV) $ sudo apt-get install yum
    (VENV) $ sudo apt-get install --reinstall xdg-utils
    ```

<br>

## 安裝 screen

> 為了讓服務持續運作，screen 可以在終端機關閉後背景持續執行

- install

    ```bash
    (VENV) $ sudo apt-get install screen
    ```

- 建立 screen

    ```bash
    (VENV) $ screen -S robot
    ```

- 會建立一個新的終端機，若要退出使用 `ctrl+a+d`

- 在新終端機進入虛擬環境

    ```bash
    $ source VENV/bin/activate #進入虛擬環境
    ```

<br>

## git

> 把程式 clone 下來吧

- install

    ```bash
    (VENV) $ sudo apt-get update
    (VENV) $ sudo apt-get install git
    ```

- 設定你的帳戶

    ```bash
    (VENV) $ git config --global user.name "cleo"
    (VENV) $ git config --global user.email "cleo@gmail.com"
    ```

- use

    ```bash
    (VENV) $ git clone https://github.com/cleoliu/你的專案.git
    (VENV) $ git init #初始化.git
    (VENV) $ git fetch origin
    (VENV) $ git status
    (VENV) $ git pull
    ```

<br>

## 啟用 Robot

- 執行以下啟用

    ```bash
    (VENV) $ cd Chat_Robot
    (VENV) Chat_Robot $ python Robot.py
    ```

- 掃 QRcode 登入唄
- 如果不把　screen　裡的程式給結束都會持續運作的

<br>

***

## 疑難雜症 Q&A

- UnicodeEncodeError: ‘cp950’ codec can’t encode character

    ```bash
    $ chcp 65001
    ```

- yum except KeyboardInterrupt，e: 错误

    ```bash
    [root@localhost bin]# yum
    File "/usr/bin/yum"，line 30
    except KeyboardInterrupt，e:
                ^
    SyntaxError: invalid syntax
    ```
    因為 yum 是用 python2 的語法，解法為指向以前的舊版本

    ```bash
    $ sudo vim /usr/bin/yum
    ```

    ```python
    #!/usr/bin/python
    import sys
    try:
        import yum
    ```

    將上面的語句改為：

    ```python
    #!/usr/bin/python2.6
    import sys
    try:
        import yum
    ```

<br>

***
## ref

- [Google Cloud Platform架站(2) - 新增虛擬機(VM)](http://robarter.pixnet.net/blog/post/223284352-%5Bgcp%5Dgoogle%E9%9B%B2%E7%AB%AF%E6%9E%B6%E7%AB%99%282%29---%E6%96%B0%E5%A2%9E%E8%99%9B%E6%93%AC%E6%A9%9F%28vm%29)

<br><br>