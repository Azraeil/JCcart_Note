04:35
## Step.2 路由設定

完整的code

`config/routes.rb`
```rb
Rails.application.routes.draw do

  resources :statics, :only => [:index]
  root "statics#index"

  resources :items, :only => [:index, :show]

  namespace :dashboard do
    resources :orders
    namespace :admin do  # 不該用 admin 關鍵字
      resources :items  
      resources :cates  
      resources :orders
      resources :users
      resources :managers # 要有s, JC大補打得時候漏掉
    end
  end

end

```
我們會分三層

**第一層：public**
首頁
不需要新增，刪除，這些行爲不該在首頁發生
>public是給所有使用者

```
resources :statics, :only => [:index]
root "statics#index"
```

**第一層：購物車**
商品 items
商品列表 index
商品顯示 show
```
resources :items, :only => [:index, :show]
```

**第二層：dashboard**

>dashboard是給一般使用者

**第三層：admin**

>admin是給管理者

一個系統在打時，建議從後面打到前面，所以我們先從管理介面`:admin`開始打
因爲管理界面的View會比前面的使用者多
```ruby
namespace :dashboard do  #第二層 消費者(買家)購物車
  resources :orders # 訂單

  namespace :admin do    #第三層 管理者(賣家)頁面
    resources :items  # 要賣的東西
    resources :cates  # 要賣的東西的分類
    resources :orders # 訂單
    resources :users  
    resources :managers
  end
end
```
namespace ... do ... end, 大概類似一個資料夾

namespace 裡面與外面的code，他的差異在於**路由**

namespace 裡面的網址：`dashboard/items/action`

namespace 外面的網址：`/statics/action`

由於我們現在是教學示例，所以admin還是寫`:admin`，但你實際在做商品時，不該這樣寫，會讓你的後台容易被猜到，應該改用亂碼寫，像是`:sakwejh`，所以做商品時，後台路由其實該長這樣
```ruby
namespace :dashboard do
  resources :orders

  namespace :sakwejh do  

    resources :items  # 要賣的東西
    resources :cates  # 要賣的東西的分類
    resources :orders # 訂單
    resources :users
    resources :managers
  end
end
```
#接下來是註冊系統
