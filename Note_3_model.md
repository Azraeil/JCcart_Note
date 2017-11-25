## Step.4 models
<br>
這部分跟 Note 2 開頭我們的設定有關係
#### Manager model

edit `app/models/manager.rb`

```rb
class Manager < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable
  #, :registerable,  #由於管理員不允許被註冊，所以把這段砍掉
  #  :recoverable, :rememberable, :trackable, :validatable(驗證)
end
```

<br>
#### User model

edit `app/models/user.rb`

```rb
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
  # 注意：上面的逗點要註解掉或是刪掉哦，不然之後 migrate 會有 error
         #:recoverable, :rememberable, :trackable, :validatable
end
```

<br>
#### Item model

***
##### 注意！！！(此爲筆記作者註解)
如果要用下指令快速建model，model name後面一定要有欄位名字與型態，例如
```rb
rails g model Item name:string
```
***
如果只打 `rails g model Item` 就會有error

在此我們不用以往下指令快速建model的方式，使用JC的手刻寫法

我們一樣去[paperclip github](https://github.com/thoughtbot/paperclip)，看到**Models**那邊，於是我們給`item.rb`加上

edit `app/models/item.rb`
以下是斷行後的結果：
```rb
class Item < ActiveRecord::Base
  has_attached_file :cover,
      styles: {
        original: "1024x1024>", #預設就有，就用吧
        medium: "300x300>",
        thumb: "100x100>"
      },  #style很重要，取決於你要幾張圖片，下面有文章連結講解
      default_url: "/images/:style/missing.png"
  validates_attachment_content_type :cover, content_type: /\Aimage\/.*\z/
end

```

style很重要，取決於你要幾張圖片，[RailsFun的這篇文章](http://railsfun.tw/t/paperclip/64)就是在講這個

<br>
觀察 db/schema.rb 發現：
```rb
  create_table "items", force: true do |t|
    ...
    裏面沒有分類："cate"
  end
```

由於JC忘了在Item加一個分類`cate`所以再寫一個新的migration
```rb
rails g migration add_item_cate
```

add to `db/migrate/XXXXXX_add_item_cate.rb`
```rb
add_column :items, :cate_id, :integer, :null => false
```
然後`rake db:migrate`

<br>
接著我們回到`item.rb`，加上
```rb
class Item < ActiveRecord::Base
  belongs_to :cate #這行還要多研究 跟DAY2影片有關

  has_attached_file :cover,
  ...
end
```

<br>
#### Cate model

create  `app/models/cate.rb`
```rb
class Cate < ActiveRecord::Base
  has_many :items # 單數還是複數？？JC好像是單數

  # 有興趣還可以研究 acts_as_tree, 這是一個 gem, 樹狀結構
end
```

<br>
#### Order model

create `app/models/order.rb`
```rb
class Order < ActiveRecord::Base
  has_many :order_items # 單數還是複數？JC好像是單數
end
```

<br>
#### OrderItem model

create `app/models/order_item.rb`
```rb
class OrderItem < ActiveRecord::Base
  belongs_to :item
  belongs_to :user
  belongs_to :order #缺一不可
end
```

<br>
#### 記得先進入console測試剛剛建好的model

```rb
rails c

:001 > Order.count
:002 > Order
:003 > Item
:004 > OrderItem
:005 > User
:006 > Manager
:007 > exit
```

一開始要先`count`，再下Model Name，這樣就能知道這個table是否存在
