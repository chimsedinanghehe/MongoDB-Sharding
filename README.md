# MongoDB Replica Set trên Kubernetes (Phần 1)

## Mục tiêu
Triển khai MongoDB với cơ chế **Replica Set** trên Kubernetes để đảm bảo:
- **High Availability** (khả năng chịu lỗi cao)
- **Tự động chuyển đổi pod khi có sự cố**
- Ứng dụng vẫn hoạt động bình thường khi một pod gặp lỗi

---

## 1. Cấu trúc Deployment

Chúng ta có 3 pod MongoDB chạy trong chế độ Replica Set:

![Các pod dành cho replica set](https://github.com/user-attachments/assets/b7124a4e-77f5-420f-afd0-3581fd46217a)

Các pod này được quản lý bởi **StatefulSet**, đảm bảo:
- Tên pod cố định
- PersistentVolume gắn liền với từng pod
- Tự động tái tạo pod khi bị xóa hoặc gặp sự cố

---

## 2. Thử nghiệm khôi phục khi pod chính (Primary) sập

### Bước 1: Xem các pod hiện tại
![Danh sách pod](https://github.com/user-attachments/assets/4cf9dd41-74ad-42fe-8176-fa07a844167f)

### Bước 2: Xóa pod Primary
![Xóa pod Primary](https://github.com/user-attachments/assets/9304080a-4934-4cef-9cad-1e5900f9ae9d)

### Bước 3: Kiểm tra trạng thái
- Kubernetes sẽ **tự động chuyển Primary sang pod Secondary** mới (`my-mongodb-1`).

![Pod Secondary được nâng lên Primary](https://github.com/user-attachments/assets/e4720696-d6bc-412f-875a-392a40f08e4e)

- Khoảng **30s sau**, pod cũ (`my-mongodb-0`) được tạo lại và Replica Set trở về trạng thái đầy đủ.

![Pod cũ được tạo lại, app vẫn hoạt động bình thường](https://github.com/user-attachments/assets/a6700f8f-a480-4063-ba5a-a994c1e8c333)

### Kết quả
- Ứng dụng vẫn hoạt động bình thường
- Không có downtime nhờ cơ chế Replica Set và StatefulSet

---

## 3. Lưu ý
- Mỗi pod cần **PersistentVolume riêng**, đảm bảo dữ liệu không bị mất khi pod bị xóa
- Thời gian chuyển đổi Primary có thể vài giây, ứng dụng cần retry connection nếu cần
