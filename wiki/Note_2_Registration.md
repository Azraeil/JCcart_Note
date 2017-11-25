## Step.3? 註冊系統
09:00
<br><br>

使用`devise`這個gem，[devise GitHub](https://github.com/plataformatec/devise)
看 Readme, 第一個要做的就是在 Gemfile ：

gem 'devise'
先在我們的專案裝好 devise
```
rails g devise:install
```
JC發現有問題，debug 開始 10:40
發現是打錯字 XD
再下一次指令，應該要成功
成功之後，觀察一下 log，devise 有些注意事項會講
有一個是說使用者註冊驗證信件的網址
(devise github 也有這個訊息)

接著我們需要 devise 的 User model 以及 Manager model
```ruby
rails g devise User
rails g devise Manager
```

12:47 發現 devise 建 model 有問題
到 Gemfile 補上 `gem 'mysql2'` (會發現已經就有了)

發現又有問題
這時候會發現筆記作者的 Note_1 先測試是對的 XD
但是我是跑第一次，所以還是記錄一下，不然世界線不一樣
不一定要一次到位

rails console, 也會發現問題
gem list 後發現 mysql 有非常多的版本：
mysql2 (0.4.0, 0.3.20, 0.3.18, 0.3.16)
然後猜測新版本有問題
這時候編輯Gemfile指定mysql版本
`gem 'mysql2' , '0.3.20'`
0.3.20 這版本是挑 gem list 第二個版本
先刪除 Gemfile.lock
接着 bundle install
這時候 執行 rails console 就會正常沒出現 error message
***
19:00 mysql2 debug 結束

```rb
rails g devise User  
rails g devise Manager
```
`rails g devise User`
這個指令會創造一個
migration_devise_create_users.rb
models/user.rb

`rails g devise Manager`
同理
migration_devise_create_managers.rb (記得會有s)
models/manager.rb

接下來就能針對 User 跟 Manager 做一些事情

去修改`db/migrate/20160906083955_devise_create_users.rb`
JC解釋了這些功能：
`Database authenticatable`：要不要讓使用者透過資料庫登入
`Recoverable`：要不要讓使用者重置密碼
`Rememberable`：要不要讓使用者能夠記住帳號/密碼？
`Trackable`：要不要追蹤登入次數，上次與這次的登入IP

然後把`Recoverable`、`Rememberable`、`Trackable`、`Confirmable`、`Lockable`
以及下面這行都註解掉沒使用到
```rb
  # add_index :users, :reset_password_token, unique: true
```
這樣Users就會有最基本的登入功能：
帳號：email
密碼：encrypted_password

接著去修改`XXXXXXX_devise_create_managers.rb`
一樣註解掉上面那些東西。
可能就會保留追蹤登入狀態而已

migration file 改完之後：
21:30
JC 遇到一個情況是他沒有先 `rake db:create`？
`rake db:migrate`
可以一起下`rake db:create db:migrate`
這樣就會把 :users 跟 :managers 的 table 創好

<br><br>
### 建立shop的table 22:00

```rb
rails g migration init_shop
```

`vim migrate/XXXXXX_init_shop.rb`
```rb
class InitShop < ActiveRecord::Migration
  def change
    create_table :cates do |t|
      t.string :name
      t.integer :position
      t.timestamps
    end

    create_table :items do |t|
      t.integer :status, :limit => 1, :default => 0, :null => false
      t.string :name
      t.integer :price
      t.text :descript
      t.timestamps
      t.timestamp :delete_at #當你刪掉一筆訂單時，你並不是真正刪掉，只是讓數字少1，他會保留你原始的資料，方便你追帳
    end

    create_table :orders do |t|
      t.integer :user_id
      t.timestamps
      t.integer :status, :limit => 1, :null => false, :default => 0
      t.integer :total, :default => 0, :null => false
    end

    create_table :order_items do |t|
      t.integer :order_id, :null => false
      t.integer :item_id, :null => false
      t.integer :user_id, :null => false
      t.integer :price, :null => false
    end
    add_index :order_items, [:order_id] #所有可能用到WHERE的都要加到add_index裡去
  end
end
```

##### 待補充：要解釋這些table (作者忘記補了吧...這邊蠻重要的)
22:00
```rb
def change
  #我們預設商品會在某個分類底下：
  create_table :cates do |t|
    t.string :name # 分類名稱
    t.integer :position # 可以指定不同的排序方法
    t.timestamps
    # t.timestamps 就是
    # t.timestamp :create_at 與
    # t.timestamp :updated_at 的簡稱
  end
  create_table :items do |t| # items 就是我們要上架的商品
    t.string :name # 商品名稱
    t.integer :price # 商品價格
    t.string :descript # 商品敘述
    t.timestamps
    t.timestamp :delete_at # 軟刪除
    #這能保存商品的歷史記錄，刪除的時候不是真的刪掉，只是讓商品在某個時間點之後變成 "" 空的，這樣要追帳才查得到，不然真的刪光就沒搞頭了
  end
end
```

繼續接着上面的code(一樣在 change method 裏面)
訂單處理：
```rb
#訂單：
#這裏可能要考慮對應的問題：一個使用者可以一次下幾個訂單
create_table :orders do |t|
  t.integer :user_id
  t.timestamps
  t.integer :status , limit => 1 , :null => false , :default => 0
  # 目前訂單的狀態，limit => 1，當成一個 byte
  t.integer :total , default => 0 , :null => false
end
#訂單裏的購物清單：
create_table :order_items do |t|
  t.interger :order_id , :null => false #重點，誰對應到誰
  t.integer :item_id , :null => false
  t.integer :user_id , :null => false
  t.integer :price , :null => false
end
```

28:30
開始回顧整個 migration 看有沒有缺東西：
```rb
#items 通常還會加
create_table :items do |t|
  t.integer :status , :limit => 1 , :default => 0 , :null => false
  ...
end
# 通常上product的時候，也要把 index 建好，舉例：
# 用陣列
# 所有可能會被 where 到的 都要加 index
# 誰跟誰有關聯到的 column 都要加：:item_id, :user_id,

add_index :order_items , [:order_id]
add_index :items , [:serial] , :unique => true #舉例，JC後來拿掉
#上面這行的意思就是防止拿到相同的序號，支援很多種欄位
```
記憶一下：
要產生哪些 table ？
cates, items, orders, order_items

接著 `rake db:migrate` 把 tables 建出來

<br>
31:10
### 產品圖片：gem paperclip

去github頁面看用法：
使用[paperclip github](https://github.com/thoughtbot/paperclip)

paperclip 要用到 imagemagick 所以先安裝：
```
brew install imagemagick
```
裝完後可以用`convert --help`這指令問ImageMagick有什麼東西可以用
imagemagick 功能是圖片轉換

觀察 Migrations 要做的事：
JC解釋了 **Migration** 的部分
如果有個圖片要加：
```rb
def up
  add_attachment :users, :avatar
  # 前面 :users 就是 model 名稱，我們的items
  # 後面 :avatar 就是 你要秀出去的檔案名稱
end
```

接著去`Gemfile`加入ImageMagick`gem "paperclip", "~> 5.0.0"`，由於我們剛剛已經裝過，所以這行可省略
<!-- 筆記作者這行應該是買個保險，因爲之前的 Gemfile 我們沒有指定版本 -->

### 建立產品圖片 migration
```
rails g migration add_item_cover
```

然後把`XXXXXXX_add_item_cover.rb`修改成[paperclip github](https://github.com/thoughtbot/paperclip)，**Migration** 的寫法

`db/migrate/20160906094234_add_item_cover.rb`
```rb
class AddItemCover < ActiveRecord::Migration
  def up
    add_attachment :items, :cover
    # :cover 決定你要秀幾張圖片
    # 如果是相簿式，你可能要做一(產品)對多(照片)的表
  end

  def down
    remove_attachment :items, :cover
  end
end
```

然後再一次`rake db:migrate`
JC有稍微講上面那段code做了哪些事：
`vim db/schema.rb`
schema 是資料庫的快照, 資料庫的原始狀況
主要加了四個column：
t.string  "cover_file_name"
t.string "cover_content_type"
t.integer "cover_file_size"
t.datetime "cover_updated_at"

JC：Railsfun 已經有文章說明：[傳送門](http://railsfun.tw/t/paperclip/64/17)
是一長串的討論
這邊時間不夠不詳述怎麼改
