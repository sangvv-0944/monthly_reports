Ở phần trước chúng ta đã học được cách sử  dụng câu query đầu tiên như thế nào, hôm nay tiếp tục 1 kiểu request nữa đó là: Mutations

Bài trước: https://github.com/sangvv-0944/monthly_reports/blob/master/ruby%20on%20rail%20and%20graphql%20p1.md

## Vậy mutations Graphql là cái gì?

Theo docs thì nó được định nghĩa như sau

> Most discussions of GraphQL focus on data fetching, but any complete data platform needs a way to modify server-side data as well.

Mutations Graphql cho phép chúng ta có thể thêm, sửa, xóa dữ liệu trên database, kết hợp với Query để đọc dữ liệu, thì chúng ta đã có đầy đủ CRUD actions. Xong hết phần này chúng ta có thể làm được ứng dụng nho nhỏ rồi =)).

Trong bài này chúng ta vẫn sử dụng project trước và tương tác với bảng Order thông qua mode Order.

Let's go!!!

## Graphql mutations format
Cũng giống như Queries, một đoạn Graphql mutations sẽ trông giống như thế này.

```ruby
mutation {
  createOrder(input: {
    description: "Iphone X",
    total: 21
  }) {
    order {
      id
      description
      total
      payments {
        id
        amount
      }
      paymentsCount
    }
    errors
  }
}
```
Một số điểm cần chú ý trước khi tìm hiểu sâu hơn:
- Ở mỗi mutations ta sẽ định nghĩa matation {} (ở query thì sẽ là query {})
- Tiếp theo là createOrder đây là tên mutations của chúng ta cấu trúc như nhau createOrder(input: {})
- Mutations chấp nhận input là 1 hash, chúng ta có thể  pass 1 chuổi string cho description và float cho `total` (chúng ta có thể bắt buộc các trường bắt buộc hoặc không bắt bắt buộc ở file mutations)
- Tiếp theo là phần {}. Ở trong này chúng ta sẽ thêm 1 hash outline dữ liệu chúng ta muốn trả về từ mutation. Nó cũng khá giống với query, nhưng ở đây chúng ta có thể thấy nó có thể trả về errors. Phần return này là không bắt buộc nhưng chúng ta vẫn nên trả về dữ liệu mong muốn để chi tiết kết quả.

Bắt đầu thôi

## Thêm mới ở mutations

Nếu bạn theo dõi bài viết phía trước thì chúng ta đã tạo 1 file base mutation `/app/graphql/mutations/`. Nếu bạn chưa có hãy xem lại bài trước
- Ở đây chúng ta sẽ tạo mutations createOrder, nhưng chú ý ở đây sẽ tên file vẫn sẽ theo snake_case theo chuẩn ruby nha.

```ruby
# /app/graphql/mutations/create_order.rb
class Mutations::CreateOrder < Mutations::BaseMutation
    argument :description, String, required: true
    argument :total, Float, required: true

    field :order, Types::OrderType, null: false
    field :errors, [String], null: false

    def resolve(description:, total:)
        order = Order.new(description: description, total: total)
        if order.save
            {
                order: order,
                errors: []
            }
        else
            {
                order: nil,
                errors: order.errors.full_messages
            }
        end
    end
end
```
Giờ chúng ta sẽ bắt đầu phân tích file này nhé

## Đầu tiên là arguments
```ruby
# /app/graphql/mutations/create_order.rb
    argument :description, String, required: true
    argument :total, Float, required: true
```
Nếu chú ý thì ở trên đầu query chúng ta có phần `input {}` trong input các key sẽ được định nghĩa ở đây:
Ví dụ:
- Tên của argument `description`: Nó được sử dụng cho input
- Kiểu dữ liệu(String): Ở đây nếu mutation request sẽ bắn ra lỗi nếu data gửi lên không đúng.
- required, kiểu dữ liệu là boolean (ỏ đây là required: true) thì là trường này là bắt buộc, nếu là `false` thì có thể có hoặc không.

## Tiếp theo sẽ là fields
```ruby
# /app/graphql/mutations/create_order.rb
    field :order, Types::OrderType, null: false
    field :errors, [String], null: false
```
Ở đây chúng ta sẽ định nghĩa 2 fields mà mutations request sẽ returns. Ở đây chúng ta có 2 field là `order` và `errors`. Ở đây chúng ta sẽ handle 2 trường hợp:
- Nếu request mutations thành công, thì sẽ trả về object Order ( Types::OrderType nó là Query ở bài trước đó trong file `/app/graphql/types/`).
- Nếu thất bại sẽ lỗi sẽ được trả về trong này với kiểu là string
- Trường null: false ở đây cho phép trả về giá trị `nil`, nếu là true sẽ throw an errors

## Magic resolve()

```ruby
# /app/graphql/mutations/create_order.rb
  def resolve(description:, total:)
    order = Order.new(description: description, total: total)
    if order.save
      {
        order: order,
        errors: []
      }
    else
        {
          order: nil,
          errors: order.errors.full_messages
        }
    end
  end
```

Tất cả các mutations đều có method `resolve()` nó sẽ xử lý và trả về ở đây.
Đi qua một lượt nào:
- `resolve()` sẽ có các tham số ở phần `arguments` chúng ta định nghĩa phía trên.
- Tiếp theo chúng ta vẫn sử dụng ActiveRecord của rails để lưu xuống database (quá quen rồi phải không =)))
- Tiếp theo là hàm save quá tường minh rồi đúng không? Nếu thành công trả về order trả về array lỗi trống, nếu fail sẽ trả về order `nil` và arrays errors trong `errors`.
Ở trong này chúng ta cũng tạo được các helper function và gọi lại trong `resolve()`(Như ở controller bình thường).

## Thêm routing
Mặc định khi generate thì file `mutation_type.rb` sẽ trả về hello word, hãy xóa hết đi nào
```ruby
# /app/graphql/types/mutation_type.rb
module Types
  class MutationType < Types::BaseObject
  end
end
```
Giờ định nghĩa lại 1 field như sau
```ruby
# /app/graphql/types/mutation_type.rb
module Types
  class MutationType < Types::BaseObject

    field :create_order, mutation: Mutations::CreateOrder

  end
end
```
## Xong rồi thì test thôi =))
Sẽ thử lại Query đầu tiên nào
Kết quả:
![screencapture-localhost-3000-graphiql-2020-05-10-20_49_13](https://user-images.githubusercontent.com/54126514/81501172-08328380-9301-11ea-9681-d617d3cc4320.png)

Trường description required nhưng gửi lên sẽ báo lỗi như sau:
![screencapture-localhost-3000-graphiql-2020-05-10-20_50_08](https://user-images.githubusercontent.com/54126514/81501174-0963b080-9301-11ea-8345-d3add7c4f353.png)

## Tổng kết
Ở đây chỉ có create còn thiếu update và delete, nhưng mình đã giải thích khá chi tiết rồi, tự triển tiếp phần tiếp theo =)).

Link repo: https://github.com/sangvv-0944/graphql-rails-example

Bài viết tiếp theo sẽ về authenticate trong rails graphql
