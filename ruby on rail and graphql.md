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
- Write and excute vài câu query nào

GraphQL có 2 cách để tương tác với cơ sở dữ liệu
- 1. **Query**: Cho phép chúng ta get data (giống như Read, trong CRUD đó)
- 2. **Mutation**: Cho phép chúng ta thay đổi dữ liệu , như thêm mới, sửa dữ liệu hay xóa dữ liệu (giống các method còn lại Create Update Destroy trong REST)

Bắt đầu tìm hiểu nào (go)(go)
## Create new rails API app
```
rails new new_app --api
```
Quá quen thuộc rồi đúng không :)) Hãy thêm vài flag để xóa nhưng thứ không cần thiêt nào:
```
$ rails new demo-graphql-ruby-api --skip-yarn --skip-sprockets --skip-coffee --skip-javascript --skip-turbolinks -d mysql --api

```
