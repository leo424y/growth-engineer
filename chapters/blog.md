寫一個部落格
===

過去曾經寫過一系列的文章，試圖去鼓勵更多的人去寫文章。從畢業前的《[成為筆桿子](https://www.phodal.com/blog/think-of-rework-be-a-writer/)》、《[寫作驅動學習
](https://www.phodal.com/blog/write-driven-learning/)》、《[重新思考部落格的意義
](https://www.phodal.com/blog/rethink-why-write-blog/)》，到工作後的《[如何提高影響力](https://www.phodal.com/blog/how-to-improve-impact/)》，都在試圖去向人們展示寫部落格的可以促進自己學習、提高自己的影響力。

等到工作以後，發現想寫的人是沒有時間，不想寫的人不存在時間。人們陷入生活的怪圈，越是加班越沒有時間學技術，越是沒有時間學技術越是需要加班。

我尚不屬於那些技術特別好的人——我只是廣度特別廣，從拿電烙鐵到所謂的大資料。不過相比於所謂的大資料，我想我更擅長於焊電路板，笑~~。由於並非畢業於計算機專業，畢業前的實習過程中，我發現在某些特殊領域的技術比不上科班畢業的人，這意味著需要更多的學習。但是後來受益於工作近兩年來從沒有加班過，朝九晚六的生活帶來了大量的學習時間。在這個漫長的追趕過程中，我發現開發部落格相關的應用帶來了很大的進步。

我的部落格
---

### 現在，我的部落格是如何工作的？

#### HTTP伺服器

當你開發在網頁上訪問我的部落格的時候，你可能會注意到上面的協議是HTTPS。

![blog-mobile][1]

但是並不會察覺到它是HTTP2.0。而這需要一個可以支援HTTP2.0的HTTP伺服器，在不改變現在程式配置的情況下，你需要重新編譯你的HTTP伺服器。在這裡，我的部落格用的是Nginx，所以它在還只是試驗版的時候，就已經被編譯進去了。為了隱藏伺服器的版本，還需要在編譯的時候做了些手腳。除此，為了瀏覽器上的那個小綠鎖，我們還需要一個HTTPS證書，並在Nginx上配置它。

在這時，我們還需要配置一個快取伺服器。過去，我在上面用過Varinsh、Nginx Cache。儘管對於個人部落格來說，可能意義不是很大，但是總需要去嘗試。於是用到了ngx_pagespeed，它會幫我們做很多事，從合併CSS、JS，到轉圖片轉為webp格式等等。

Nginx對請求轉發給了某個埠，就來到了WSGI。

#### WSGI

接著，我們就來到了Web伺服器閘道器介面——是為Python語言定義的Web伺服器和Web應用程式或框架之間的一種簡單而通用的介面。現在，你或許已經知道了這個部落格是基於Python語言的框架。但是在我們揭曉這個答案之前，我們還需要介紹個小工具——New Relic。如果你在Chrome瀏覽器上使用Ghosty外掛，你就會看到下面的東西。

![New Relic][2]

New Relic是一個網站監測工具，Google Analytics是一個分析工具。但是，不一樣的是New Relic需要在我們啟動的時候加進去：

```
nohup /PATH/bin/newrelic-admin run-program /PATH/bin/gunicorn --workers=2 MK_dream.wsgi -b 0.0.0.0:8080 --timeout=300&
```

現在這個請求總算來到了8080埠，接著到了Gunicorn的世界裡，它是一個高效的Python WSGI Server。

過了上面幾步這個請求終於交給了Django。

#### Django

Django這個天生帶Admin的Web框架，就是適合CMS和部落格。這並不意味著它的工作範圍只限於此，它還有這麼多使用者:

![Who Use Django][3]

請求先到了Django的URL層，這個請求接著交給了View層來處理，View層訪問Model層以後，與Template層一起渲染出了HTML。Django是一個MTV框架(類似於MVC之於Spring MVC)。接著，HTML先給了瀏覽器，瀏覽器繼續去請求前端的內容。

它也可以用Farbic部署哦~~。

#### Angluar & Material Design Lite vs Bootstrap & jQuery Mobile

這是一個現代瀏覽器的前端戰爭。最開始，部落格的前端是Bootstrap框架主導的UI，而移動端是jQuery Mobile做的(PS: Mezzanine框架原先的結構)。

接著，在我遇到了Backbone後，響應了下Martin Folwer的**編輯-釋出分離模式**。用Node.js與RESTify直接讀取部落格的資料庫做了一個REST API。Backbone就負責了相應的Detail頁和List頁的處理。

儘管這樣做的方式可以讓使用者訪問的速度更快，但是我相信沒有一個使用者會一次性的把技術部落格看完。而且我部落格流量的主要來源是Google和百度。

然後，我試著用Angular去寫一些比較特殊的頁面，如[全部文章](https://www.phodal.com/all/)。但是重寫的過程並不是很順暢，這意味著我需要重新考慮頁面的渲染方式。

最後，出現了Material Design Lite，也就是現在這個醜醜的頁面，還不相容新IE（微信瀏覽器）。

作為一個技術部落格，它也用到了HighLight.js的語法加亮。

#### API

在構建SPA的時候，做了一些API，然後就有了一個Auto Sugget的功能：

![Auto Suggest][4]

或者說，它是一個Auto Complete，可以直接藉助於jQuery AutoComplete外掛。

或許你已經猜到了，既然我們已經有部落格詳情頁和列表頁的API，並且我們也已經有了Auto Suggestion API。那麼，我們就可以有一個APP了。

#### APP

偶然間發現了Ionic框架，它等於 = Angluar + Cordova。於是，在測試Google Indexing的時候，花了一個晚上做了部落格的APP。

![Blog App][5]

我們可以在上面做搜尋，搜尋的時候也會有Auto Suggestion。上面的登出意味著它有登入功能，而Hybird App的登入通常可以借用於JSON Web Token。即在第一次登入的時候生成一個Token，之後的請求，如發部落格、建立事件，都可以用這個Token來進行，直到Token過期。如果你是第一次在手機上訪問，也許你會遇到這個沒有節操的廣告：

![Install Phodal Blog App][6]

然並卵，作為我的第七個Hybird應用，它只發布在Google Play上——因為不需要稽覈。

隨後，我意識到了我需要將我的部落格推送給讀者，但是需要一個渠道。

###微信公眾平臺

藉助於Wechat-Python-SDK，花了一個下午做了一個基礎的公眾平臺。除了可以查詢最新的部落格和搜尋，它的主要作用就是讓我發我的部落格了。

![Phodal QRCode][7]

對了，如果你用Python寫程式碼，可以試試PyCharm。除了WebStorm以外，我最喜歡的IDE。因為WebStorm一直在與時俱進。


### 技術組成

So，在這個部落格裡會有三個使用者來源，Web > 公眾號 > App。

在網頁上，每天大概會400個PV，其中大部分是來自Google、百度，接著就是偶爾推送的公眾號，最後就是隻有我一個人用的APP。。。

> Web架構

伺服器：

1. Nginx(含Nginx HTTP 2.0、PageSpeed 外掛)
2. Gunicorn(2 Workers)
3. New Relic(效能監測)

DevOps:

1. Farbic（自動部署）

Web應用後臺：

1. Mezzaine（基於Django的CMS）
2. REST Framework (API)
3. REST Framework JWT (JSON Web Token)
4. Wechat Python SDK
5. Mezzanine Pagedown （Markdown

Web應用前臺:

1. Material Design Lite (使用者)
2. BootStrap (後臺)
3. jQuery + jQuery.autocomplete + jquery.githubRepoWidget
4. HighLight.js
5. Angluar.js
6. Backbone (已不維護)

移動端:

1. Ionic
2. Angular + ngCordova
3. Cordova
4. highlightjs
5. showdown.js(Markdown Render)
6. Angular Messages + Angular-elastic

微信端:

1. Wechat-Python-SDK

資料分析與收集
---

### Google Analytics & WebMaster

### APM: New Relic
