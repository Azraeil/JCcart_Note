## Step.1 環境設定

#### 建立專案
```ruby
rails new JCcart -d mysql -T
```

`-d mysql` 代表資料庫 database 選擇用MySQL
`-T` 代表不要用rails內建的單元測試

#### 修改database的帳號密碼
`vim config/database.yml`
我先前設定的MySQL密碼：**iamgroot**
要用時請改成你自己的MySQL密碼
共有兩個地方要修改

##### JC在影片教學中的寫法，會噴掉 (原因不明)
推測跟 mysql 安裝時設定的帳號密碼有關係
Rails 要去用 linux mysql 的服務，所以要帳號密碼
PS：JC後來也發現會噴掉，不過JC是把defalut的username改成`root`

`vim config/database.yml`
```ruby
default: &default
  adapter: mysql2
  encoding: utf8
  pool: 5
  username: iamgroot
  password: iamgroot
  host: localhost

  ...

  production:
    <<: *default
    database: JCcart_production
    username: iamgroot
    password: iamgroot

```

##### 我自己修改測試後，能work的寫法
```ruby
default: &default
  adapter: mysql2
  encoding: utf8
  pool: 5
  username: root
  password: iamgroot
  host: localhost

  ...

  production:
    <<: *default
    database: JCcart_production
    username: JCcart
    password: iamgroot

```

#然後Gemfile把MySQL改成`gem 'mysql2', '0.3.20'`
(這是已知影片12:47 debug 後的成果)
#### 會用到的gem

`Gemfile`
關(註解)掉`turbolink`、`jbuilder`、`sdoc`、`spring`

安裝
```ruby
gem 'will_paginate' # 分頁
gem 'awesome_print' # debug ap 指令
gem 'rails-pry'     # 擴充，取代irb
gem 'devise'        # 註冊登入系統
gem 'paperclip'     # 檔案上傳
```
存檔
然後`bundle install`(把GEM裝到完)
***
[提前測試] 建議現在做，這是筆記作者自己加在這的
用意是避免影片12:47, 開始出現的 bug
`rake db:create`，`rails console`，有問題的話這兩個就會出錯
最後`rails s`測試`localhost:3000`看是否能 work

應該將這步驟列爲標準流程，在這階段就解決掉
不現在做之後再debug會中斷思緒
容易忘了接下來要做的事(然後產生新的bug)
