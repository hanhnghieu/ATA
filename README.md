# ATA Explorer (Spring Boot + PostgreSQL), Airbus A320

Mô hình 3D máy bay minh họa theo A320. Bấm **Tách rời**, bấm vào từng bộ phận để xem chương ATA, thành phần, thông số, bảo dưỡng và an toàn. Có hai chế độ:

- **Học**: giải thích và tra cứu chương ATA.
- **Kỹ sư**: thêm ô tìm kiếm theo số chương hoặc mã (ví dụ `32-41-00`), danh sách tài liệu cần tra (AMM, TSM, IPC, SRM, WDM, MPD, MEL/CDL) và **nhật ký hư hỏng** gắn với từng chương ATA.

## Yêu cầu
- JDK 17 trở lên
- PostgreSQL 12 trở lên
- Eclipse IDE for Enterprise Java (đã có Maven/m2e). Máy cần có Internet để tải thư viện Maven và thư viện three.js.

## Chạy
1. Tạo database rỗng (pgAdmin hoặc psql):
   ```sql
   CREATE DATABASE ata_explorer ENCODING 'UTF8';
   ```
2. Mở `src/main/resources/application.properties`, sửa `spring.datasource.username` và `spring.datasource.password` cho khớp máy bạn.
3. Eclipse: **File > Import > Maven > Existing Maven Projects**, chọn thư mục này. Nếu cập nhật từ bản cũ: chuột phải project > **Maven > Update Project**, chọn Force Update.
4. Chuột phải `AtaExplorerApplication.java` > **Run As > Java Application**.
5. Mở http://localhost:8080

Mỗi lần khởi động, `schema.sql` tạo lại các bảng chương và `data.sql` nạp lại dữ liệu mẫu. **Bảng `defect` (nhật ký hư hỏng) không bị xóa** khi khởi động lại. Khi bạn bắt đầu tự sửa dữ liệu chương trong DB, đổi `spring.sql.init.mode=never`.

## API
- `GET /api/chapters`, `GET /api/chapters/{số chương}`: dữ liệu chương ATA.
- `GET /api/defects?chapter=32`: danh sách hư hỏng (bỏ `chapter` để lấy tất cả).
- `POST /api/defects`: ghi nhận hư hỏng mới.
- `PATCH /api/defects/{id}`: cập nhật trạng thái, biện pháp xử lý, tài liệu tham chiếu.

Quy tắc đang áp dụng: đóng hoặc hoãn hồ sơ phải có biện pháp xử lý hoặc lý do; hồ sơ đã đóng không sửa được; không có thao tác xóa hồ sơ.

## Cấu trúc
```
src/main/java/com/example/ata
  model/       AtaChapter, Spec, Maintenance, Defect, DefectSeverity, DefectStatus
  repository/  AtaChapterRepository, DefectRepository
  service/     AtaChapterService, DefectService (kiểm tra dữ liệu, quy tắc đóng hồ sơ)
  dto/         ChapterDto, DefectDto, DefectRequest, DefectUpdate
  web/         AtaChapterController, DefectController
src/main/resources
  schema.sql, data.sql     bảng và dữ liệu mẫu 18 chương
  static/index.html        giao diện 3D (three.js)
```

## Chưa có và cần làm tiếp nếu dùng thật
- **Đăng nhập và phân quyền.** Hiện ai mở được trang đều ghi được hồ sơ, tên người ghi nhận do tự gõ. Nhật ký này chưa thay được hồ sơ kỹ thuật chính thức của hãng khai thác.
- **Dữ liệu A320 chính thức.** Nội dung chương trong `data.sql` là kiến thức chung về A320, tự soạn, không lấy từ AMM, MPD hay tài liệu Airbus. Nhãn chu kỳ (hằng ngày, A-check, C-check) chỉ để học. Không có số task AMM thật nào trong dữ liệu. Muốn dùng cho công việc thật phải đối chiếu và thay bằng dữ liệu hiệu lực từ Airbus hoặc hãng khai thác, theo đúng MSN và revision.
- **Mô hình 3D** dựng từ hình khối cơ bản, chưa phải bản vẽ A320 (ví dụ có 2 đầu pitot, A320 thật có 3; chưa dựng slat và trim ngang).
