<h1 align="center">Giới thiệu về CI/CD</h1>

CI/CD là tập hợp các quy trình và công cụ giúp tự động hóa việc xây dựng, kiểm thử và triển khai phần mềm. Đây là một trong những thành phần cốt lõi của DevOps và thường được sử dụng cùng Docker, Kubernetes, git và các nền tảng Cloud.

CI/CD là một bộ đôi công việc, bao gồm CI (Continuous Integration) và CD (Continuous Deployment/Delivery), là quá trình tích hợp (integration) thường xuyên, nhanh chóng hơn khi code cũng như thường xuyên cập nhật phiên bản mới (delivery).

**CI/CD là gì**

Continuous Integration (Tích hợp liên tục) là quá trình dev thường xuyên merge code vào nhánh chính. Mỗi lần thay đổi, hệ thống sẽ tự động lấy source code mới, build ứng dụng, chạy unit test và build artifact như docker image hay JAR. Nếu có lỗi thì sẽ dừng quá trình và thông báo lỗi.

Continuous Delivery/Deployment (Cập nhật liên tục) là quá trình triển khai phiên bản mới của code sau khi quá trình CI đã hoàn thành. Khi có phiên bản mới của code, nó sẽ được build và test ở môi trường deploy, chuẩn bị để đẩy lên chính thức. 

- Đối với continous delivery, cần có sự xác nhận thủ công trước khi code được đẩy lên live
- Continous deployment thì tự động hóa luôn cả bước này, đẩy mọi thay đổi code lên live mà không cần sự xác nhận thủ công

CI/CD pipeline là một chuỗi hoàn chỉnh kết hợp 2 khái niệm này, tự động thực hiện từ khi dev push code cho đến khi ứng dụng chạy trên server:

```
Developer

git push
      │
      ▼
Github
      │
      ▼
Build
      │
      ▼
Unit Test
      │
      ▼
Integration Test
      │
      ▼
Build Docker Image
      │
      ▼
Push Image Registry
      │
      ▼
Deploy Kubernetes
      │
      ▼
Health Check
      │
      ▼
Chạy ứng dụng
```

**Những lợi ích của CI/CD**

- Giảm xung đột khi merge code, phát hiện lỗi sớm.
- Luôn có phiên bản source code build được.
- Tự động hóa quá trình deploy thay vì build thủ công, giảm sai sót.
- Đảm bảo tính nhất quán.
- Dễ rollback nếu cần.
- Hỗ trợ DevOps, thống nhất quy trình, theo dõi lịch sử triển khai, tăng khả năng hợp tác giữa các nhóm.