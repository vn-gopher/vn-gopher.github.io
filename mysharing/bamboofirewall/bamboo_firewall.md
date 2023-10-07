---
layout: default
---
# Đặt vấn đề
Trong suốt thời gian làm việc với vai trò là SystemAdmin hoặc DevOps Engineer cho các hệ thống dưới on-premise tôi gặp rất nhiều vấn đề liên quan tới việc làm sao để thiết lập chính sách bảo mật cho các máy chủ Linux mình quản lý. Hầu hết các cách mọi người đang làm là triển khai iptables hoặc 1 giải pháp firewall mềm nào đó để đảm bảo an toàn thông tin.

Việc sử dụng iptables để thiết lập chính sách cho server và vùng máy chủ gặp phải vô vàn vấn đề trong việc quản trị:

- Tôi phải tạo 1 file hoặc script để nạp rule cho từng máy chủ tôi quản lý
- Rule iptables sẽ phải khai báo theo ip, port, subnets,.. và gắn với từng máy chủ
- Rất là phức tạp nếu phải viết rule cho 1 supper app với số lượng máy chủ CSDL và dịch vụ lên tới con số hàng trăm node.
- Rất là cực nhọc khi tôi vừa mở rule cho phép ip nguồn của app x.x.x.[1-22] gọi sang ip đích của db y.y.y.y port đích 3306 giao thức TCP. Bây giờ server y.y.y.y có kế hoạch đổi sang ip mới z.z.z.z (lưu ý rule này cần mở cho hàng trăm node app). Tôi phải upde lại nội dung file hoặc script và apply lại trên từng đó node (có thể chạy ansible playbook) - nhưng khoai.


Từ các vấn đề trên tôi nghĩ nếu việc quản trị và thiết lập rule firewall cho các máy chủ dưới on-premise cũng giống như cách mà GCP (Google cloud platform), AWS Cloud Computing Services,... đang thiết lập cho các zone mạng, máy chủ trên cloud thì thật tuyệt. Khi đó ta sẽ có:

- Policies as Code (provision rule iptables theo code, giống như terraform provision...)
- Các node mạng và network, subnet được đánh label và việc khai báo rule sẽ based on trên lable. Do đó khi tôi thay đổi địa chỉ IP của database server sẽ không phải thay đổi gì về rule (đơn giản vì rule map với lable rồi)
- Apply nhanh và hiệu lực toàn miền cho hàng nghìn hàng triệu máy chủ 1 cách nhanh chóng. Và tôi không phải chạy ansible playbook nào cả
- Đảm bảo rule chặt chẽ và chính xác, cấp vừa đủ quyền cho dịch vụ, đối tượng mạng

# Miêu tả giải pháp



# Mục lục
- [Mô hình thiết kế](#principle)
- [Cài đặt và cấu hình](#scalability)
- [Các câu hỏi thường gặp](#availability)