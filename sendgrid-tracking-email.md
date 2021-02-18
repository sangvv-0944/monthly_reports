# Sử dụng Sendgrid để Tracking Email bằng Ruby on Rails

Việc sử dụng email để gửi các thông tin về dịch vụ, marketing...Ở Rails thì đã support ActionMailer setup rất nhanh qua SMTP Gmail, SendGrid để send mail. Có bao giờ bạn tò mò người dùng đã xem email hay chưa, hay đã click vào liên kết chưa? hay click bao nhiêu lần?..., để xây dựng tracking như vậy thì rất khó dùng trick các kiểu, nhưng ở **SendGrid** đã support chúng ta còn hơn thế nữa ^^

Ở SendGrid sử dụng 2 cách để tracking Email:
1. Sử dụng **Webhook** event
2. Sử dụng API để query

## Bắt đầu thôi
Tạo mới một ứng dụng rails
```sh
 rails new sendgrid-tracking-example
```
Add Gemfile
```
gem "sendgrid-actionmailer"
```

Chạy `bundle install`
Tiếp tục thêm vào `config/application.rb`
```ruby
config.action_mailer.delivery_method = :sendgrid_actionmailer
config.action_mailer.sendgrid_actionmailer_settings = {
  api_key: ENV['SENDGRID_API_KEY'],
  raise_delivery_errors: true
}
```

## Xây dựng 1 Action Mailer để gửi mail
```sh
bundle exec rails generate mailer Message
```
Khi chạy xong rails sẽ tạo file `app/mailers/message_mailer.rb`, mở lên vào viết 1 method để xử lý send mail trong này

```ruby
class MessageMailer < ApplicationMailer
  def email
    @message = params[:message]
    mail(
      to: @message.to,
      subject: @message.subject,
      from: ENV['FROM_EMAIL'],
      custom_args: {
        id: @message.id
      }
    )
  end
end
```

Phần này chúng ta có thể thêm custom_args để phía dưới tìm id của mail và update status

## Email template
Tạo file view `app/views/message_mailer/email.html.erb`, nội dung đơn giản như sau:

```
<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
    <%= simple_format(@message.body) %>
  </body>
</html>
```

## Gửi mails
```
  def create
    @message = Message.new(message_attributes)
    if @message.save
      MessageMailer.with(message: @message).email.deliver_now!
      redirect_to root_path
    else
      render :new
    end
  end
 ```
 ## Sử dụng webhook event để nhận trạng thái của event
 ```
 bundle exec rails generate controller webhooks
```
Ở controller webhooks ta sẽ nhận event khi mail có người nhận được
```
class WebhooksController < ApplicationController
  def create
    events = params["_json"]
    events.each do |event|
      next unless event["id"]
      # Update status mail
      message = Message.find(event["id"])
      message.update_attribute(:status, event["event"])
    end
    render json: {status: :ok}
  end
end
```

## Add routing

```
  resources :messages, only: [:new, :create, :show]
  get '/messages', to: redirect('/')
  post '/webhooks', to: 'webhooks#create'
  root to: 'messages#index'
```

## Test ngay thôi
Để sử dụng được webhook bạn cần deploy lên ngrok hoặc heroku để hoạt động. Ở đây chỉ sử dụng code để test, bạn có thể sử dụng Action Cable để realtime status, còn thực tế thì sẽ dùng background job để update status vì số lượng request lên webhook rất lớn. hoặc sử dụng api của SendGrid để query metrics và update. Đọc tham khảo tại đây: https://sendgrid.api-docs.io/v3.0/email-activity/filter-all-messages
Cảm ơn bạn đã đọc bài :D
