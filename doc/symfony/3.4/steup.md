## 安裝 & 設定 Symfony 框架

此篇文章會解釋各種安裝 Symfony 的方式，以及如何解決大部分在安裝過程中發生的問題。

### 產生 Symfony 應用程式

Symfony 提供了一個專用的指令「Symfony Installer」來簡化 Symfony 應用程式的建立。該指令相容於 PHP 5.4 並只需要在你的系統中安裝一次：

```terminal
# Linux and macOS systems
$ sudo mkdir -p /usr/local/bin
$ sudo curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony
$ sudo chmod a+x /usr/local/bin/symfony

# Windows systems
c:\> php -r "file_put_contents('symfony', file_get_contents('https://symfony.com/installer'));"
```

```note
在 Linux 和 MacOS 會產生 `symfony` 這個全域指令。而在 Windows 下，移動 `symfony` 檔案到系統環境變數 `PATH` 的資料夾下，並建立一個 `symfony.bat` 的檔案使其成為全域的指令，或是將此檔案移動到你覺得適合的資料夾下。

Windows 的環境變數 `PATH` 包含多個資料夾，每個資料夾下的指令都會成為全域的指令

~~~terminal
# for example, if WAMP is used ...
c:\> move symfony c:\wamp\bin\php
# create symfony.bat in the same folder
c:\> cd c:\wamp\bin\php
c:\> (echo @ECHO OFF & echo php "%~dp0symfony" %*) > symfony.bat
# ... then, execute the command as:
c:\> symfony

# moving it to your projects folder ...
c:\> move symfony c:\projects
# ... then, execute the command as
c:\> cd projects
c:\projects\> php symfony
~~~
```

當你的「Symfony Installer」已安裝完成，使用 `new` 這個指令產生你的第 1 個 Symfony 應用程式

```terminal
$ symfony new my_project_name 3.4
```

該指令會產生一個新資料夾 `my_project_name/`，包含了最新的 Symfony 穩定版本。此外 Installer 會檢查你的系統是否滿足執行 Symfony 應用程式所需的系統需求，若是無法滿足，你會看到一組修正清單來滿足系統需求。

```note
如果 Installer 無法運作或是看不到任何輸出文字，請確認 [PHP Phar extension](https://php.net/manual/en/intro.phar.php) 有被安裝在你的電腦上。
```

```note
如果 SSL 憑證沒有正確安裝在你的系統上，你可能會看到這個錯誤：

> cURL error 60: SSL certificate problem: unable to get local issuer certificate.

你可以用以下的方法解決這個問題：

1. 從該網址下載最新的憑證清單 [https://curl.haxx.se/ca/cacert.pem](https://curl.haxx.se/ca/cacert.pem)
2. 將下載的 `cacert.pem` 儲存在你的系統中
3. 更新 `php.ini` 檔，設定 `cacert.pem` 的路徑
    ~~~ini
    ; Linux and macOS systems
    curl.cainfo = "/path/to/cacert.pem"
    
    ; Windows systems
    curl.cainfo = "C:\path\to\cacert.pem"
    ~~~
```

### 開發時指定 Symfony 版本

如果你的專案需要指定 Symfony 版本時，可以使用 `new` 指令的第二個參數：

```terminal
# 使用最新的版本在任何的 Symfony 分支
$ symfony new my_project_name 2.8
$ symfony new my_project_name 3.1

# 使用指定的 Symfony 版本
$ symfony new my_project_name 2.8.3
$ symfony new my_project_name 3.1.5

# 使用 beta 或 RC 版本 (在測試 Symfony 新版本時很有用)
$ symfony new my_project 2.7.0-BETA1
$ symfony new my_project 2.7.0-RC1

# 使用最新的 'lts' 版本 (Long Term Support version 長期支援版本)
$ symfony new my_project_name lts
```

每個版本都有各自 *專屬* 文件，你可以在任何一個文件頁面上做切換。

```note
閱讀 [_Symfony Release process_](https://symfony.com/doc/3.4/contributing/community/releases.html) 可以更了解為何 Symfony 有著許多的版本，以及選擇那個版本用於你的專案上。
```

### 使用 Composer 建立 Symfony 專案

如果你無法使用 Symfony Installer 的話，你可以使用 [_Composer_](https://getcomposer.org/) 來建立 Symfony 專案，這是現代 PHP 專案在使用的套件管理工具。

如果你還沒有安裝 Composer 在你的電腦上，請先照 [_installing Composer globally_](https://symfony.com/doc/3.4/setup/composer.html) 的文件安裝 Composer，然後執行 `create-project`  的指令安裝一個最新穩定版本的 Symfony 專案。

```terminal
$ composer create-project symfony/framework-standard-edition my_project_name
```

你也可以用 `create-project` 指令的第二個參數，來安裝其他的 Symfony 版本：

```terminal
$ composer create-project symfony/framework-standard-edition my_project_name "3.4.*"
```

```tip
如果你的網路速度很慢，你可能會覺得 Composer 沒有在運作，在 `composer create-project` 指令中使用 `-vvv` 的選項，來顯示 Composer 在處理的所有細節。
```

### 運行 Symfony 專案

在生產環境的伺服器上，Symfony 使用 Apache 或 Nginx 這類的 Web Server (請見 [configuring a web server to run Symfony]())。然而，在你本地端開發的機器上，你也可以使用 Sumfony 提供的 Web Server (使用 PHP 內建的 Web Server)。

首先，[安裝 Symfony Web Server](https://symfony.com/doc/3.4/setup/built_in_web_server.html)，然後執行這個指令：

```terminal
$ cd my_project_name/
$ php bin/console server:run
```

打開你的瀏覽器並訪問 `http://localhost:8000/` 網址，可以見到 Symfony 的歡迎頁面：

![Demo Picture with Symfony 3.4](/upload/article/1/72/7258b0e28d83b6eba9ecf68960d0b348.png)

如果你沒看到歡迎頁面，而是看到空白頁或錯誤的頁面，這是資料夾權限設定錯誤造成的。解決這個問題的方法在「[Setting up or Fixing File Permissions](https://symfony.com/doc/3.4/setup/file_permissions.html)」有做解說。

當你結束你 Symfony 專案的開發後，在終端機或是 command console 中輸入Ctrl+C 來停止
 Web Server。

```tip
Symfony 的 Web Server 對開發來說是很棒的，但不應該用於生產環境上。改成使用 Apache 或 Nginx，請見[設定 Web Server](https://symfony.com/doc/3.4/setup/web_server_configuration.html)。
```

### 檢查 Symfony 專案的設定和安裝環境

Symfony Installer 會檢查你的系統是否已經可以運行 Symfony 專案。然而，在 console 下的 PHP 設定會與 Web Server 的設定不同。因此，Symfony 提供了一個視覺化的設定檢查工具。訪問以下的網址來檢查你的設定，並在進行下一步之前修正相關問題。

```
http://localhost:8000/config.php
```

### 修正檔案權限問題

如果你有任何檔案權限錯誤或是看到空白畫面，請見 [Setting up or Fixing File Permissions](https://symfony.com/doc/3.4/setup/file_permissions.html) 來取得更多資訊。

### 更新 Symfony 專案

到這邊你已經建立了一個有完整功能的 Symfony 專案！每個 Symfony 專案依賴一些第三方套件，存放於 `vendor` 資料夾，並使用 Composer 來管理。

定期地更新這些套件是很好的作法，可以避免一些臭蟲或是安全漏洞。執行 `update` 的 Composer 指令來一次更新全部的套件 (這會需要花費些許分鐘來完成，具體花費的時間看你專案套件依賴的複雜程度而定)：

```terminal
$ cd my_project_name/
$ composer update
```

```tip
Symfony 提供一個指令來檢查你專案使用的套件是否含有任何已知的安全性漏洞：

~~~terminal
$ php bin/console security:check
~~~

定期地執行該指令是一個安全的作法，儘快更新套件或是更換成其他適合的套件。
```

### 安裝 Symfony Demo 版本或其他發行版本

你已經下載了 [_Symfony Standard Edition_](https://github.com/symfony/symfony-standard)：預設的 Symfony 初始專案，透過這份文件，你將使用該專案來開發你的應用程式！

Symfony 也有其他的初始專案範本提供您使用：

- [The Symfony Demo Application](https://github.com/symfony/demo)  
    這是個完整功能的專案，展示了 Symfony 專案的建議開發方式。該專案被認為是一個 Symfony 新手的學習工具，其原始碼包含大量的註解和有用的註解。

- [The Symfony CMF Standard Edition](https://github.com/symfony-cmf/standard-edition)  
    這是一個包含了 Symfony CMF 的初始專案，[Symfony CMF](http://cmf.symfony.com/) 可以讓開發者更簡易地加入 CMS 的功能到 Symfony 專案上。

- [The Symfony REST Edition](https://github.com/gimler/symfony-rest-edition)  
    該專案展示了如何使用 [FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle) 和其他相關的 Bundle 來建立一個提供 RESTful API 的專案。

### 安裝現有的 Symfony 專案

在一個協作開發的 Symfony 專案中，我們會將程式碼上傳到一個共享的版本庫中。

建議不要將一些檔案 ([parameters.yml](https://symfony.com/doc/3.4/configuration.html#config-parameters-yml)) 和資料夾 (`vendor/`, `cache`, `logs`) 上傳到版本庫內。

接下來，我們可以照以下的步驟來下載現有的 Symfony 專案：

```terminal
# clone 並下載專案內容
$ cd projects/
$ git clone ...

# 讓 Composer 安裝專案所依賴的套件到 vendor/ 資料夾
$ cd my_project_name/
$ composer install

# Composer 會請你輸入一些參數的數值
$ ...
```

### Keep Going!

完成了設定之後，該是時候來 [_Create your first page in Symfony_](https://symfony.com/doc/3.4/page_creation.html)

```see-also
此篇文章主要翻譯自 Symfony 官方文件，並做了些許的修改，原始文章請見<a href="https://symfony.com/doc/3.4/setup.html" target="_blank">此連結</a>
```