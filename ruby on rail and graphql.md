Trong quá trình phát triển REST API mỗi lần phía client cần sửa thêm hay bớt 1 thuộc tính nhỏ nhưng mình luôn phải sửa
trên server để đáp ứng được nhu cầu. Hoặc cùng 1 api dùng chung nhưng bên này chỉ dùng những trường này, chổ khác lại sử dụng trường khác dễ 
dẫn đến thừa dữ liệu => giảm tải dữ liệu tăng performance
 Mấy năm trở lại đây xuất hiện GraphQL khắc phục những nhược điểm phía trên.
 Vậy GraphQL là gì ?
 GraphQL là một tiêu chuẩn API mới cung cấp một giải pháp hiệu quả, mạnh mẽ và linh hoạt hơn thay thế cho REST.

Nó đã được phát triển bởi Facebook và hiện nay được duy trì bởi một cộng đồng lớn của các công ty và cá nhân từ khắp nơi trên thế giới.

Cốt lõi của GraphQL là cho phép client có thể xác định chính xác dữ liệu cần lấy về từ server.

Thay vì nhiều endpoints trả về cấu trúc dữ liệu cố định, một máy chủ GraphQL chỉ đưa ra một endpoint duy nhất và đáp ứng chính xác dữ liệu mà khách hàng yêu cầu.
