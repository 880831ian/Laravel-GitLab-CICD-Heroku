# 部署 Laravel 於 Heroku 搭配 GitLab CI/CD

經過上一篇文章 [如何從頭打造專屬的 GitLab CI/CD](https://pin-yi.me/gitlab-cicd/) 的學習，讓我們了解到 GitLab CI/CD 的整個流程，接著我們本次要把 Laravel 給部署到 Heroku 透過 GitLab 的 CI/CD 去達成，不需要透過任何人工去測試，並上架程式到 HeroKu 上，全部都依賴 GitLab CI/CD，讓我們接著看下去吧！

<br>

當然，此文章程式碼也會同步到 Github ，需要的也可以去查看歐！要記得先確定一下自己的版本 [Github 程式碼連結](https://github.com/880831ian/Laravel-GitLab-CICD-Heroku) 😏

## 版本資訊

* macOS：11.6
* Docker：Docker version 20.10.14, build a224086
* Laravel Installer：2.3.0 
* Laravel Framework：9.14.1
* gitlab.com：GitLab Enterprise Edition 15.1.0-pre

<br>

首先，我們第一步驟就是先建立一個 Laravel 專案，至於為什麼要選擇用 Laravel 來當作 GitLab CI/CD 的範例呢？因為 Laravel 內建有 PHPUnit 的測試腳本，可以讓我們在 CI 測試時，更好的展現 CI 的功能！，有關於 Laravel 相關內容，這邊一樣推薦兩篇文章給大家閱讀：🤣

* [Laravel 介紹 (使用 Laravel 從零到有開發出一個留言板功能並搭配 RESTful API 來實現 CRUD)](https://pin-yi.me/laravel/)
* [Laravel 進階 (內建會員系統、驗證 RESTful API 是否登入、使用 Repository 設計模式)](https://pin-yi.me/laravel-advanced/)

又工商了一波 XD

<br>

## 建立 Laravel 專案

請大家依照 Laravel 官方文件來建立 Laravel 環境，也可以看小弟我的文章拉 👆👆👆，請記得要先安裝好 php 以及 composer，接著按照以下步驟來建立。

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/create.png)

<br>

這時候瀏覽 `http://127.0.0.1:8000`，如果都正確，應該會看到 Laravel 的首頁

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/laravel.png)

<br>

## 測試本地 Unit Test

接著我們剛剛有提到選用 Laravel 的原因是 Laravel 有 PHPUnit 單元測試可以使用，所以我們現在先在本地端來測試 Unit Test，專案預設有放一個單元測試在 `tests/Unit/ExampleTest.php`。我們先再次確認環境是否有安裝好，再來執行單元測試。

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/unittest.png)

<br>

執行後，應該都會是通過的畫面，如下圖：

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/demo_test.png)

<br>

## GitLab CI 建置

### 上傳 Laravel 專案

接下來我們要上傳含有 Unit Test 專案到 GitLab 上，步驟如下，如果已經熟悉如何將專案推到 GitLab，可以直接跳到 [在 GitLab 上執行單元測試](#在-gitlab-上執行單元測試)

1. 在 [gitlab.com](https://gitlab.com) 上點選建立專案，選擇 **Create blank project**，也可以直接瀏覽該網址 `https://gitlab.com/projects/new#blank_project`。
2.   輸入專案名稱可以選擇 Project deployment target 為 **Heroko**，選擇 Public，最後按下 **Create project**

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/gitlab.png)

<br>

3. 於專案資料夾下加入 remote 遠端 GitLab，並 Push 將專案推上去。

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/gitlab_push.png)

<br>

成功推上去，可以到 GitLab 上，看到我們剛剛的專案！

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/gitlab_show.png)

<br>

### 在 GitLab 上執行單元測試

要在 GitLab 上執行 CI/CD 就需要有 Runner，這次我們選擇使用 [gitlab.com](https://gitlab.com) 的 Shared runners，想要使用 Specific runners，可以查看上一篇 [如何從頭打造專屬的 GitLab CI/CD](https://pin-yi.me/gitlab-cicd/#%E8%87%AA%E6%9E%B6-runner-specific-runners) 文章

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/runner.png)

<br>

接下來在專案的根目錄撰寫我們的 `.gitlab-ci.yml` 檔案，之後再次上傳 GitLab，當我們根目錄有此檔案，GitLab-CI 就會讀取並依照內容啟動 Runner 來執行工作：

* .gitlab-ci.yml
```yml
image: lorisleiva/laravel-docker:latest

Unit_test:
  before_script:
    - composer install --prefer-dist --no-ansi --no-interaction --no-progress --no-scripts
  script:
    - ./vendor/bin/phpunit --testsuit Unit --coverage-text --colors=never
```

<br>

說明一下這個 `yml` 檔內的設定是在做什麼：
* image：因為我們執行 CI/CD 過程中，需要有 PHP、Compose、NPM 等工具，有這些套件管理工具就可以延伸去安裝更多套件，如果一開始沒有安裝，就會很麻煩，其中一個辦法就是去 Runner 環境修改並安裝，但因為方便以及我們這次使用 Share runners，所以不能修改別人的 Runner，另一個辦法是可以使用 `image` 關鍵字，可以讓 Runner 切換到另一個環境去執行工作 (Job)，我們這邊使用 [lorisleiva/laravel-docker:latest](https://github.com/lorisleiva/laravel-docker) ，他裡面已經幫我們安裝好上述的工具了！
* Unit_test：這邊也是我們的 Job，那裡面主要是先用 composer install 去安裝我們需要的套件，最後在執行 phpunit 來做單元測試。

<br>

### 上傳 .gitlab-ci.yml

接著我們使用以下指令將含有 `.gitlab-ci.yml` 的專案上傳到 GitLab，並回到 GitLab 選擇 CI/CD，可以查看目前的 Pipelines，會有我們剛剛所新增的 Runner。

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/gitlab-ci.png)

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/pipeline.png)

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/runner-cicd.png)

<br>

### 設置須通過測試才可以合併

當我們有了測試還不夠，要怎麼確保每隻要上線 (合併到主分支) 的程式都有經過測試才上線呢？

接下來我們可以在 GitLab 裡面做這些設定，先到專案的 Setting → General → Merge requests → Merge checks 點選 **Pipelines must succeed**：

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/mergechecks.png)

<br>

### 測試是否可以阻擋未成功情況

我們先模擬要開發新功能，所以在 master 最新 commit 下，建立一個新分支 `new`

```sh
git checkout -b "new"
```

<br>

接著修改單元測試，故意新增錯誤的測試，開啟專案的 `tests/Unit/ExampleTest.php`，最下面加上紅色框框程式碼：

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/exampletest.png)

`assertEquals` 會檢查這兩個值是否相同，不同的話，就會跳出錯誤，所以我們故意輸入 1 和 2。
<br>

並將它上傳到 GitLab，並發出 Merge Request 看看會有什麼結果！

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/gitlab-test.png)

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/mergerequest.png)

<br>

可以看到我們合併在 Pipeline 測試時，因為 new 沒有通過測試，所以也沒有辦法進行合併！

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/mergeerror.png)

<br>

## GitLab CD 建置

我們玩完 CI 後，接著要把程式部署到伺服器或是雲端上，這時候我們不需要透過人工手動的方式，只需要有 CD 來幫我們自動化部署就可以拉！如果不太清楚，可以參考這張圖片：

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/gitlab_workflow.png)
(圖片來源：[GitLab](https://docs.gitlab.com/ee/ci/introduction/))


當我們剛剛進行 CI 的整合測試，最後經過 Review and approve 合併到主分支，這時候如果我們有設定 CD，CD 就會幫我們部署到服務上，我把 CD 流程轉成文字步驟說明：

<br>

1. 把新功能分支合併到 master 分支，代表功能已經可以上線
2. GitLab 觸發 Gitlab-CI 執行 pipeline
3. Gitlab-CI 執行自動化測試
4. Gitlab-CI 測試成功後，執行部署到正式伺服器
5. 回傳執行結果至 GitLab

<br>

那想要達成自動化部署之前，必須能在遠端用指令下達部署更新！簡單來說有兩件事情：
1. 要先整理再更新專案時需要哪些指令，並將其寫成腳本
2. 需要獲得伺服器的授權，可以對伺服器下達更新專案的腳本

<br>

我們以現在 Laravel 專案來說，套用上面講的兩件事情：
1. 腳本製作：上線新版本大概要執行以下圖片的內容

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/laravel_up.png)

<br>

2. 對遠端伺服器下指令：通常使用 ssh 與 伺服器做溝通，所以先在伺服器產生授權金鑰給要遠端控制的電腦，如果要給 Gitlab-CI 控制的話，也需要把金鑰存在 GitLab 上，通常使用 `ssh user@remote.server 'git pull'` 來下達更新專案的指令

<br>

本篇我們要部署的是 PaaS 的 HeroKu，可以減少時間去架設環境，就可以達到我們想要的效果，那接著會帶大家從 Heroku 設定開始歐！先簡單介紹一下 Heroku：

Heroku 是一個支援多種程式語言的雲平台即時服務(PaaS)， 是一種雲端運算服務，提供運算平台與解決方案服務，PaaS提供使用者將雲端基礎設施部署與建立至使用者端，或者藉此獲得使用程式語言、程式庫與服務。使用者不需要管理與控制雲端基礎設施（包含網路、伺服器、作業系統或儲存），但需要控制上層的應用程式部署與應用代管的環境。

<br>

### 創建 Heroku 專案

那要使用 Heroku 當然需要一組帳號拉，建立帳號我應該不用再多介紹了吧 🤡 &nbsp;我們直接到 Heroku 頁面，右上角 New，點選 Create new app，輸入本次專案名稱，我就取叫 `laravel-gitlab-cicd-heroku` (這個不能與別人重複，因為他會生成專屬網頁)， 進去後，點選右上角有一個 **Open app**，就會跳出這個專案專屬的網頁：
 	
<br>

{{< image src="/images/laravel-gitlab-cicd-heroku/herokuapp.png"  width="1000" caption="Heroku 專屬網頁" src_s="/images/laravel-gitlab-cicd-heroku/herokuapp.png" src_l="/images/laravel-gitlab-cicd-heroku/herokuapp.png" >}}

<br>

### 設定 HeroKu 與 GitLab 連線

先點選右上角個人頭像 → Account setrtings → 在 Account 往下滑 → API Key，點選 **Reveal** 並將該 API 記住，這是等等透過 GitLab 部署時會用到的 API Token：

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/token.png)

<br>

回到 GitLab 專案底下，Settings → CI/CD → Variables，他可以將變數設定在這邊，再讓 `.gitlab-ci.yml` 來抓取變數，設定以下兩個變數：(詳細可以參考[官網](https://docs.gitlab.com/ee/ci/variables/index.html#add-a-cicd-variable-to-a-project))

1. Key 名稱(HEROKU_PRODUCTION_PROJECT_NAME)，Value 值(設定我們剛剛在 Heroku 部署的專案名稱，我的是 `laravel-gitlab-cicd-heroku`)
2. Key 名稱(HEROKU_PRODUCTION_API_KEY)，Value 值(這個就是我們上面的 API Key，每個人都要用自己的歐！上面的我已經重設了 😎 &nbsp;)

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/variables.png)

* 這邊要注意先把預設的 Protect variable 給關閉，他預設會只能在受保護的分支或標籤運行，但我們這此以簡單為主，所以這些設定都先關掉。

<br>

### 新增 Heroku 識別檔案

接下來我們要新增一個檔案名為 `Procfile`，它是 Heroku 部署更新時會啟動的對象，注意他沒有副檔名，我們在裡面輸入以下：(我們使用合併後的 master)

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/procfile.png)

它代表我們網頁服務使用 apache2 指令運行並把入口指向專案資料夾中的 laravel 專案的入口資料夾。

<br>

### 修改 .gitlab-ci.yml

我們修改原本用來 CI 的腳本，來設定自動化部署的任務 Job 及腳本

* .gitlab-ci.yml
```
image: lorisleiva/laravel-docker:latest

Production_Deploy:
  stage: Production_Deploy
  before_script:
    - apk add ruby ruby-dev ruby-irb ruby-rake ruby-io-console ruby-bigdecimal ruby-json ruby-bundler yarn ruby-rdoc >> /dev/null
    - apk update
    - gem install dpl >> /dev/null
  script:
    - dpl --provider=heroku --app=$HEROKU_PRODUCTION_PROJECT_NAME --api-key=$HEROKU_PRODUCTION_API_KEY
```

<br>

最後上傳 GitLab 來觸發 Gitlab-CI 執行自動化部署 (上傳指令就不多說囉，想必大家都會了吧！，不會的話可以去看 [Git 介紹](https://pin-yi.me/git/)，裡面有詳細的介紹 😍 &nbsp;)

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/cd_pass.png)

<br>

可以看到部署成功，我們也來看看 Runner 運作狀況：

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/cd_runner.png)

看到他成功將服務給部署到 `https://laravel-gitlab-cicd-heroku.herokuapp.com/`。

<br>

既然已經部署好了，當然要去看一下我們的網頁啊，但當我們打開部署好的網頁，會發現跳出 500 Error，雖然他與我們 CI/CD 沒有關係，但我們還是試著解決，那這個問題會發生是因為我們沒有給環境變數的 APP_KEY，這個 Key 可以在專案的 .env 取得，拿到後開啟 Heroku → Setting → Config vars 將 APP_KEY 設定上去。

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/key.png)

<br>

最後重新整理 `https://laravel-gitlab-cicd-heroku.herokuapp.com/`，就可以看到我們部署上去的網站囉！

<br>

![圖片](https://raw.githubusercontent.com/880831ian/Laravel-GitLab-CICD-Heroku/master/images/laravel_show.png)

<br>

## 參考資料

[Gitlab-CI 入門實作教學 - 單元測試篇](https://nick-chen.medium.com/gitlab-ci-%E5%85%A5%E9%96%80%E7%AD%86%E8%A8%98-%E5%96%AE%E5%85%83%E6%B8%AC%E8%A9%A6%E7%AF%87-156455e2ad9f)

[Gitlab-CI 自動化部屬部署](https://medium.com/@nick03008/%E6%95%99%E5%AD%B8-gitlab-ci-%E5%85%A5%E9%96%80%E5%AF%A6%E4%BD%9C-%E8%87%AA%E5%8B%95%E5%8C%96%E9%83%A8%E7%BD%B2%E7%AF%87-ci-cd-%E7%B3%BB%E5%88%97%E5%88%86%E4%BA%AB%E6%96%87-cbb5100a73d4)

[部署 Laravel 於 Heroku 搭配 Gitlab CI/CD](https://medium.com/@vip131430g/%E9%83%A8%E7%BD%B2-laravel-%E6%96%BC-heroku-%E6%90%AD%E9%85%8D-gitlab-ci-cd-6d59a66aebdb)
