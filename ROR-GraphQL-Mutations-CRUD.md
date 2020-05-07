Ở phần trước chúng ta đã học được cách sử  dụng câu query đầu tiên như thế nào, hôm nay tiếp tục 1 kiểu request nữa đó là: Mutations

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
    description: "Octo Octa - Resonant Body (vinyl)",
    total: 21.82
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
