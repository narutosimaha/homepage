+++
author = "Vu Tran"
title = "Scalable blog design system (Vi)"
date = "2022-05-16T23:27:55+07:00"
description = "Blog system designed for serving massive readers (10k tps)"
tags = [
    "scale",
    "blog"
]
categories = [
   "design"
]
aliases = ["migrate-from-jekyl"]
+++

# Thiết kế hệ thống blog phục vụ cho lượng lớn người đọc (10k tps)


## 1. Một số giả thiết đặt ra.

Vì dự án này đang ở mức khá trừu tượng, nên để làm rõ hơn mục tiêu cần đạt được, phần này sẽ định nghĩa thêm một số khía cạnh cần thiết liên quan đến phạm vị của dự án.

### Ngữ cảnh
Giả sử ta cần thiết kế cho một hệ thống ứng dụng web để phục vụ cho một số lượng lớn những người đọc giả, họ có thể tìm kiếm đọc blog ưa thích hoặc tự tạo một blog riêng cho mình (dựa trên account base). Một blog có thể được gán với các nhãn để phân loại chủ đề mà blog đang nói đến, thuận tiện cho việc tìm kiếm blog theo lĩnh vực quan tâm đối với các độc giả khác.

### Use cases

Tương ứng với ngữ cảnh một số use case được đặt ra liên quan đến tính scalable của hệ thống như sau.
* Một **User** có thể request để đọc và viết đến ứng dụng.
    * Hệ thống cần phải tiếp nhận (thêm, chỉnh sửa blog), lưu trữ dữ liệu (video, hình ảnh), và trả về kết quả trên yêu cầu.
* Hệ thống cần đảm bảo phục vụ cho ít nhất một triệu user. Cụ thể, thõa mãn được những yêu cầu sau:
    * 100 tỉ lần gửi đọc request trên tháng. (Hơn 10k tps)
    * Tỉ lệ số lượng request đọc trên viết là 100:1
    * 1KB cho mỗi lần viết.
* Bên cạnh đó, hệ thống cần phải có tính tin cậy để mang mức độ an toàn đến việc lưu trữ blog của người dùng.


### Một số giả định khác
* Giả sử giao thông hạ tầng mạng không được phân phối đều.


## 2. Thiết kế hệ thống.
{{< centerImage link="/self/archictect.png" caption="Ảnh 1. Blog design system">}}

## 3. Giải thích những component được sử dụng.

> Dive into details for each core component.

### Các component cần thiết
{{< centerImage link="/self/necessary.png" caption="Ảnh 2. Component cần thiết">}}

Ta sẽ có một số component cơ bản để một hệ thống web có thể hoạt động như Client browser, DNS, Database. Về database system, vì đây là một hệ thống blog đơn giản, dữ liệu từ người dùng cần lưu không có tính relational quá nhiều, hơn thế nữa vì bài toán giải quyết ở đây là mở rộng phần mềm, nên để dễ dàng hơn NOSQL database là một lựa chọn tốt cho việc hiện thực hệ thống.

Có rất nhiều hướng giải quyết cho một bài toán mở rộng trong phát triển phần mềm. Đối với dự án này, do ta chưa có cách nào để kiểm thử performance của hệ thống xem có ổn định hay không, cho nên chủ yếu tôi sẽ đưa ra nhiều hướng giải quyết nhất có thể cho đến khi cảm thấy nó đủ ổn cho việc phục vụ số lượng lớn user như đã đề ra trong mô tả. 
### Mở rộng theo chiều sâu
Mở rộng theo chiều sâu là một cách tiếp cận để giải quyết cho bài toán này. Để có thể phục vụ được một số lượng lớn user một cách nhanh chóng, cách nhanh nhất là đi mua các tài nguyên sử dụng các công nghệ hiện đại, có hiệu suất cao như CPU, memory, IO, một số thiết bị về network,... Tuy nhiên, cách này bất cập ở chỗ các thiết bị này không phải là rẻ khi ta phải mua để phục vụ cho một số lượng lớn đọc giả ngày càng tăng cho hệ thống blog hiện tại. Càng khó khăn hơn khi ta cần phải xây dựng hệ thống backup dữ liệu, để đáp ứng tính có sẵn của phần mềm.


### Mở rộng theo chiều ngang.
Một cách khác, ta có thể áp dụng bài toán mở rộng theo chiều ngang. Ở đây, thay vì phải mua một số ít thiết bị tiên tiến với giá thành cao, với cùng ngân sách ta có thể sử dụng để mua nhiều thiết bị hiệu năng thấp hơn và áp dụng một số bài toán mở rộng phần mềm trên các thiết bị đó để đổi lấy lại một giá trị tương đương. Ngoài ra, cũng có thể kết hợp hai cách trên để làm sao đó với giá cả hợp lý ta có thể đổi lại một hệ thống tối ưu nhất theo mong muốn.

#### Object storage và CDN
{{< centerImage link="/self/objectStore.png" caption="Ảnh 3. Object storage và CDN">}}


Quay lại áp dụng vào hệ thống blog, về cơ bản mục tiêu ưu tiên nhất của chúng ta là làm sao để có thể xử lý 10k transaction một second. Ta thấy, khi phải xử lý quá nhiều yêu cầu như vậy đặc biệt là yêu cầu truy vấn dữ liệu mềm như ảnh và video có độ phân giải cao, database thường có hiện tượng quá tải, để giải quyết vấn đề này việc ứng dụng object storage như S3 là điều cần thiết để có thể lưu trữ các dữ liệu không có cấu trúc như vậy, bên cạnh đó còn gia tăng thêm tốc độ truy xuất cho người dùng.

Một vấn đề nảy sinh khác khi ta truy xuất xong dữ liệu bỏ vào blog và gửi về cho người dùng ta phải đối mặt với hạn chế trong bài toán network đó là propagation delay và tranmission delay do không phải lúc nào hệ thống ta cũng ở gần với người dùng, tệ hơn nữa khi phải gửi về các file có kích thước lớn, làm giảm throughput đáng kể. Ta có thể sử dụng CDN như Amazon's CloudFront kết hợp với object storage để giải quyết vấn đề này, bằng cách đó dữ liệu của ta sẽ được đặt ở gần user hơn, hệ thống ta không cần phải lo các request đến các file như vậy nữa.

#### Database

Database nên có những tính năng như query cache đánh chỉ số index dựa theo primary key (tên tài khoản người dùng) hay secondary key (tên nhãn blog) để giúp tăng tốc hơn trong việc xử lý các câu lệnh truy vấn làm gia tăng hiệu suất hệ thống.

#### Load balancer vs Phân chia Web server và Application server
{{< centerImage link="/self/separate.png" caption="Ảnh 4. Load balancer và các server">}}

Với bài toán mở rộng theo chiều ngang, ta sử dụng khá nhiều server, và không phải lúc nào server cũng nhận được một số lượng tải như nhau, cụ thể đối với blog, do ở máy tính cá nhân người dùng có cache lại các ip address của web server cho các lần truy xuất tiếp theo tùy vào TLL (Time to live), cho nên với những người dùng đưa quá nhiều video có độ phân giải cao vào blog của họ, vô tình có thể làm cho một server nào đó bị quá tải. Load balancer có thể giải quyết vấn đề này bằng cách phân bổ đều các request đến các server. Ngoài ra,load balancer còn có thể kết thúc protocol SSL tại đây, sau đó giao tiếp nội bộ với các server. (Giảm thiểu việc set up và giải phóng một số tài nguyên). Một số load balancer có thể sử dụng: Amazon's ELB, HAProxy....

Bên cạnh đó, để giảm thiểu tải trên một server ta cần phải phân tích server ra thành hai thành phần chuyên biệt khác nhau một là web server dùng để lo cho việc tiếp nhận request, xử lý các vấn đề liên quan đến mạng như session, cookie,... Hai là application server (bussiness value) dùng để process yêu cầu của user. Điều này làm cho hai loại server này trở nên độc lập, giúp gia tăng tính bất đồng bộ làm tăng tốc độ thu hồi dữ liệu từ phía người dùng hơn.


### Về độ tin cây
{{< centerImage link="/self/database.png" caption="Ảnh 5. Về độ tin cây">}}

Ở đây, để mang đến độ uy tín cho hệ thống cũng như sự an toàn nhất định đến cho người dùng trong việc lưu trữ và quản lý các blog ta cũng cần một số hệ thống lưu trữ để thực hiện việc backup dữ liệu cho người dùng. Tôi sử đề xuất sử dụng thêm công nghệ Database master-slave replication, điều này có giúp có thể vừa backup dữ liệu nếu database master gặp hư hỏng, vừa có thể gia tăng tính avaiability khi database slave cho phép người dùng đọc được blog mặc dù không thể thêm hay chỉnh sửa blog (giúp giảm chi phí). Ngoài ra, optional ta có thể thêm một số web server làm slave (active-passive) để gia tăng thêm độ tin cậy.

## 4. Considerations.

Ngoài ra một số chức năng khác có thể thêm vào, như đánh giá, bình luận một blog, nhắn tin với các blogger nổi tiếng,... Lúc này có thể cân nhắc chuyển sang sử dụng SQL database thay vì NOSQL.

Nếu vẫn chưa đạt yêu cầu đề ra, có thể sử dụng thêm một số công nghệ caching như memcached, redis,...

Có thể cân nhắc sử dụng autoscaling để giảm hao tổn tài nguyên.

