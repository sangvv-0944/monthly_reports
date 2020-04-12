# Ruby on rails và Graphql tạo Query đầu tiên

Trong quá trình phát triển REST API mỗi lần phía client cần sửa thêm hay bớt 1 thuộc tính nhỏ nhưng mình luôn phải sửa
trên server để đáp ứng được nhu cầu. Hoặc cùng 1 api dùng chung nhưng bên này chỉ dùng những trường này, chổ khác lại sử dụng trường khác dễ 
dẫn đến thừa dữ liệu => giảm tải dữ liệu tăng performance
 Mấy năm trở lại đây xuất hiện GraphQL khắc phục những nhược điểm phía trên.
 Vậy GraphQL là gì ?
 GraphQL là một tiêu chuẩn API mới cung cấp một giải pháp hiệu quả, mạnh mẽ và linh hoạt hơn thay thế cho REST.

Nó đã được phát triển bởi Facebook và hiện nay được duy trì bởi một cộng đồng lớn của các công ty và cá nhân từ khắp nơi trên thế giới.

Cốt lõi của GraphQL là cho phép client có thể xác định chính xác dữ liệu cần lấy về từ server.

Thay vì nhiều endpoints trả về cấu trúc dữ liệu cố định, một máy chủ GraphQL chỉ đưa ra một endpoint duy nhất và đáp ứng chính xác dữ liệu mà khách hàng yêu cầu.

Hôm nay mình cùng tìm hiểu xây dựng GraphQL với Ruby on rails sử dụng gem `graphql`

# Overview
Chúng ta bắt đầu với các bước:
- Create a Rails API app
- Add some Model
- Add Graphql
- Viết thử vài câu query bằng graphql

GraphQL có 2 cách để tương tác với cơ sở dữ liệu
- **Query**: Cho phép chúng ta get data (giống như Read, trong CRUD đó)
- **Mutation**: Cho phép chúng ta thay đổi dữ liệu , như thêm mới, sửa dữ liệu hay xóa dữ liệu (giống các method còn lại Create Update Destroy trong REST)

Bắt đầu tìm hiểu nào (go)(go)
## Create new rails API app
```
rails new new_app --api
```
Nhưng ở đây mình muốn bỏ generate vài thứ râu ria không cần thiết vì mình ch viết API, và sử dụng cơ sở dữ liệu là `mysql` nhé. Nếu muốn test nhanh thì dùng `sqlite` default cũng được

```
$ rails new demo-graphql-ruby-api --skip-yarn --skip-coffee --skip-javascript --skip-turbolinks -d mysql --api

```
Sau khi init project xong, nếu sử dụng sqlite thì tiếp tục còn nếu sử dụng mysql thì nhớ config db trong file database.yml nhé.

## Bắt đầu thôi!!!
### Generating models
Tạo 2 model mẫu là Order và Payment mẫu bằng lệnh sau:
```shell
$ rails g model Order description:string total:float
$ rails g model Payment order_id:integer amount:float
```
Thiết lập relationship cho các model, ở đây 1 Order có sẽ nhiều Payment như ta sẽ định nghĩa như sau trong model
```ruby
# app/models/order.rb
class Order < ApplicationRecord
    has_many :payments
end
```
```ruby
# app/models/payment.rb
class Payment < ApplicationRecord
    belongs_to :order
end
```
Sau đó tiến hành chạy lệnh
```shell
$ rails db:create rails db:migrate
```
để thêm tạo database và thêm model vào database.
### Thêm data mẫu
```ruby
# db/seeds.rb
order1 = Order.create(description: "Iphone X", total: 100.00)
order2 = Order.create(description: "Nokia 1208", total: 200)
order3 = Order.create(description: "Vertu, total: 10)

payment1 = Payment.create(order_id: order1.id, amount: 50.00)
payment2 = Payment.create(order_id: order2.id, amount: 20.00)
payment3 = Payment.create(order_id: order3.id, amount: 5)
```
Chạy `rails db:seed` để thêm dữ liệu
## Add Graphql
```ruby
# Gemfile
gem "graphql"
```
Sau đó chạy `bundle install` đê thêm gem vào project

Chạy lệnh generate graphql: `$ rails generate graphql:install`

Sau khi chạy xong sẽ thêm thư như này
![Screenshot from 2020-04-11 23-03-55](https://user-images.githubusercontent.com/54126514/79064013-85051a00-7ccf-11ea-86a4-21f2ff01bc1d.png)

Nó sẽ add thư mục /graphql/ trong thư mục app, và tạo 1 controller trong `/controllers/graphql_controller.rb`
Và routes đã được add thêm 1 route bạn mở lên xem nhé
 ### Tạo Graphql tương ứng vớ model
 ```shell
$ rails generate graphql:object order
$ rails generate graphql:object payment
 ```
 ### Định nghĩa GraphQL Object Types
 GraphQL Types dùng để xác định các trường muốn get
 Ví dụ
 ```ruby
# app/graphql/types/payment_type.rb
module Types
    class PaymentType < Types::BaseObject
        field :id, ID, null: false
        field :amount, Float, null: false
    end
end
 ```
 Trong payment ta chỉ get 2 trường đó là id và amount, và trường id có type ID bởi vì nó là trường đặt biệt Primary Key, trường amount khi trả về kiểu dữ liệu sẽ là Float. Ở đây set null: false  thì khi Query Response về `nil` sẽ bắn ra Exception. [Xem nhiều Types describe objects ở đây](https://graphql.org/learn/schema/#type-system)
 
 Và để ý mỗi GraphQL object type đều kế thừa `Types::BaseObject` nên chúng ta có thể sử dụng được `Types::PaymentType`
 
 Giờ sẽ định nghĩa OrderType 
```ruby
# app/graphql/types/order_type.rb
module Types
    class OrderType < Types::BaseObject
        field :id, ID, null: false
        field :description, String, null: false
        field :total, Float, null: false
        field :payments, [Types::PaymentType], null: false
        field :payments_count, Integer, null: false

        def payments_count
            object.payments.size
        end
    end
end
```
Như đã nói phía trên fileds payments, chúng ta sẽ sử dụng `Types::PaymentType` để get dữ liệu từ relationship has_many-belongs_to

Ở đây chúng ta muốn custom thêm 1 file là payments_count để đếm số payments của một order, mà chúng ta không lưu trường này vào databse nên chúng ta sẽ tạo 1 method để trả về dữ liệu.
**Note chú ý:** để truy bảng relationship chúng ta sử `object` 
### Định nghĩa Query Type
Khi GraphQL nhận được một *Query request* hay *Mutation request* nó sẽ được định tuyến qua QueryType class(giống như Query object Type phía trên) để nhận về 1 số fields
 Ví dụ
```ruby
# app/graphql/types/query_type.rb
module Types
    class QueryType < Types::BaseObject
        field :all_orders, [Types::OrderType], null: false

        def all_orders
            Order.all
        end
    end
end
```
Ở đây ta đơn giản chỉ get tất cả Order qua method: `all_orders`

## Thử query nào
Chúng ta có thể sử dụng Postman hay Insomnia gửi **POST** request lên http://localhost:3000/graphql
Nhưng ở đây bạn có thể dùng giao diện trực quan  GraphiQL IDE để tương tác dễ hơn nhé có hỗ trợ complete, bằng gem `graphiql-rails`
Chạy thử câu query
```query {
    allOrders {
        id
        description
        total      
        payments {
            id
            amount
        }
        paymentsCount
    }
}
```
Và kết quả:
![screencapture-localhost-3000-graphiql-2020-04-12-15_57_46](https://user-images.githubusercontent.com/54126514/79064843-6a826f00-7cd6-11ea-963a-5da904c40e80.png)
Và đừng quên hãy bật server trước nhé `rails s`

## Tổng kết
Qua ví dụ đơn giản về sử dụng GraphQL trong Rails chúng ta chỉ get data chỉ qua 1 endpoint duy nhất đó là /graphql khi muốn thêm trường hay bỏ bớt một trường cực kí đơn giản đúng không nào?
Và đây chỉ là mới chỉ có get (Read) còn 1 phần quan trọng nữa đó thêm sửa xóa thông qua **Mutations** ở phần sau nhé.
Repo example: https://github.com/sangvv-0944/graphql-rails-example
