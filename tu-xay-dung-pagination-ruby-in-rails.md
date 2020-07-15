# Hướng dẫn xây dựng một pagination cho ứng dụng Ruby on Rails
## Đặt vấn đề
 Bạn có biết vì sao người ta lại thường chọn RoR cho các dự án startup? Ở google có vô vàng bài viết trả lời cho câu hỏi này trong đó có lý do vô cùng quan trọng đó là RoR có rất nhiều gem, support rất nhiều trong quá trình phát triển một dự án, ví dụ như authenticate thì devise, hay phân quyền thì cancancan, trong việc phân trang thì kaminary với pagy rất nổi tiếng, support view đẹp, giảm tải dữ liệu load ra. Khi cài 1 gem thì cực đơn giản và khi sử dụng đơn giản chỉ là 
 ```
 User.where(created_at: 7.days.ago).page(10).per(10)
 ``` 
 rất đơn giản đúng không?
 
 Hôm nay mình sẽ cùng tìm hiểu cách hoạt động của chúng và tự viết cho mình một pagination.

![Screenshot from 2020-07-17 22-34-22](https://user-images.githubusercontent.com/54126514/87804314-dea45400-c87d-11ea-99a5-02b7271fec7c.png)

Chúng ta có thể thấy khi sử dụng câu get dữ liệu phía trên phía sau có limit và offset.
Vậy là gem phân trang dựa vào 2 câu sql đó limit: số record mình muốn lấy, và offset là vị trí mình lấy.

Với các câu query bình thường thì nó karmari hoạt động rất tốt, nhưng khi câu query phức tạp chứa group by + thêm xóa mềm thì karminary cũng cung cấp chúng ta API cho phân trang arrays: 
```
# controller
rows = ActiveRecord::Base.connection.exec_query("SELECT complex_query FROM big_users_table JOIN ... GROUP BY ...").to_a
@users = Kaminari.paginate_array(rows)

# view
<%= paginate @users %>
```
Câu trên sẽ vẫn hoạt động tốt nếu dữ liệu ít, nhưng khi dữ liệu lên hằng 100k records phải load hết rồi đem mới lấy 10 record. Quá phí đúng không. 

Ý tưởng:
 Viết 1 helper nhận vào 1  trả về gồm:ctiveRecord::Relation và trả về dữ liệu:
 - total_pages: Tổng số page
 - total_count: Tổng số record 
 - all: Trả về số data ở page hiện tại 
Vậy đã xong phần ý tưởng

```ruby
class Paginator
  def initialize data_group_by, page, per
    @data = data
    @page = page.to_i || 1
    @per = per.to_i || 10
    @total_count = total_count
  end

  def current_page
    @page
  end

  def total_count 
    execute(to_count_sql).to_a.first["total_count"]
  end

  def total_pages
    (@total_count.to_f / @per).ceil
  end

  def all
    @data.limit(@per).offset(offset)
  end

  def offset
    (@page - 1) * @per
  end

  def execute sql
    ActiveRecord::Base.connection.exec_query(sql)
  end

  def to_count_sql
    "SELECT count(*) as total_count FROM (" << @data.to_sql << ") AS paginable"
  end
end
```
 Sử dụng:  Paginator.new(User.all.group("...."), param[:page], params[:per_page))

Các hàm được xây dựng tương tự như karminary nên chúng ta vẫn có thể sử dụng helper `paginate` ở view 1 cách bình thường.

Chúng ta sẽ count số lượng record thông qua subquery và sử dụng nó để tính các `total_pages`, `total_count`

Hi vọng bài viết có thể giúp ích cho bạn ^^


 Link tham khảo:
 https://kirshatrov.com/2015/11/08/kaminari-custom-query/
