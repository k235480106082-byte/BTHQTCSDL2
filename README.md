# BÀI KIỂM TRA SỐ 2 – HỆ QUẢN TRỊ CƠ SỞ DỮ LIỆU

## Thông tin sinh viên

- Họ và tên: Nguyễn Văn Mạnh
- Mã só sinh viên: K235480106082
- Đề tài: Quản lý kho bán điện thoại

---

# PHẦN 1: TẠO VÀ SỬ DỤNG DATABASE

## Ảnh 1: Tạo và sử dụng database

- Lệnh SQL: 

```sql
CREATE DATABASE [QLKhoDienThoai_K235480106082];
GO

USE [QLKhoDienThoai_K235480106082];
GO
```

- Mục đích: 
Tạo một database riêng để chứa toàn bộ dữ liệu quản lý kho điện thoại
Chuyển context sang database đó để làm việc
- Kết quả:
Xuất hiện database QLKhoDienThoai_K235480106082 trong SQL Server
Các lệnh tiếp theo sẽ được thực thi trong database này

<img src="picture 2/p1.png" width="100%">

---

## Ảnh 2: Tạo bảng LoaiSanPham
 
- Lệnh SQL: 

```sql
CREATE TABLE LoaiSanPham (
    MaLoai INT IDENTITY(1,1) PRIMARY KEY,
    TenLoai NVARCHAR(100) NOT NULL UNIQUE
);
```

- Mục đích:
Tạo bảng danh mục để phân loại điện thoại (bảng cha)
- Kết quả:
Có bảng LoaiSanPham
Sau này bảng DienThoai sẽ dùng MaLoai làm khóa ngoại → đảm bảo liên kết đúng

<img src="picture 2/p2.png" width="100%">

---

## Ảnh 3: Tạo bảng SanPham

- Lệnh SQL

```sql
IF OBJECT_ID('SanPham', 'U') IS NOT NULL
    DROP TABLE SanPham;

IF OBJECT_ID('LoaiSanPham', 'U') IS NULL
BEGIN
    CREATE TABLE LoaiSanPham (
        MaLoai INT IDENTITY(1,1) PRIMARY KEY,
        TenLoai NVARCHAR(100) NOT NULL UNIQUE
    );
END

CREATE TABLE SanPham (
    MaSP INT IDENTITY(1,1) PRIMARY KEY,
    TenSP NVARCHAR(100) NOT NULL,
    MaLoai INT NOT NULL,
    GiaNhap DECIMAL(12,2) CHECK (GiaNhap >= 0),
    GiaBan DECIMAL(12,2) CHECK (GiaBan >= 0),
    SoLuong INT DEFAULT 0 CHECK (SoLuong >= 0),
    CONSTRAINT FK_SP_Loai FOREIGN KEY (MaLoai)
    REFERENCES LoaiSanPham(MaLoai)
);
```

- Mục đích:
Tự kiểm tra và tạo LoaiSanPham nếu thiếu
Xóa SanPham cũ (nếu lỗi trước đó) rồi tạo lại đúng chuẩn
- Kết quả:
Không còn lỗi FOREIGN KEY
Bảng SanPham chắc chắn tạo thành công
Database vẫn giữ nguyên, không cần reset

<img src="picture 2/p3.png" width="100%">

---

## Ảnh 4: Tạo bảng DonHang

- Lệnh SQL

```sql
IF OBJECT_ID('DonHang', 'U') IS NULL
CREATE TABLE DonHang (
    MaDH INT IDENTITY(1,1) PRIMARY KEY,
    NgayDat DATE DEFAULT GETDATE(),
    TongTien DECIMAL(12,2) DEFAULT 0
);
```

- Mục đích:
Tạo bảng lưu thông tin đơn hàng
Dùng để chứa dữ liệu tổng (ngày, tổng tiền)
- Kết quả:
Có bảng DonHang
Mỗi đơn có mã tự tăng, ngày tự động, tổng tiền mặc định 0

<img src="picture 2/p4.png" width="100%">

---

## Ảnh 5: Tạo khóa ngoại

- Lệnh SQL:

```sql
IF OBJECT_ID('ChiTietDonHang', 'U') IS NOT NULL
    DROP TABLE ChiTietDonHang;

CREATE TABLE ChiTietDonHang (
    MaDH INT NOT NULL,
    MaSP INT NOT NULL,
    SoLuong INT CHECK (SoLuong > 0),
    GiaBan DECIMAL(12,2),

    PRIMARY KEY (MaDH, MaSP)
);

ALTER TABLE ChiTietDonHang
ADD CONSTRAINT FK_CTDH_DH
FOREIGN KEY (MaDH) REFERENCES DonHang(MaDH);

ALTER TABLE ChiTietDonHang
ADD CONSTRAINT FK_CTDH_SP
FOREIGN KEY (MaSP) REFERENCES SanPham(MaSP);
```

- Mục đích
Tạo bảng trước → thêm khóa ngoại sau để tránh lỗi khi tạo trực tiếp
Đảm bảo MaDH, MaSP đúng kiểu INT NOT NULL

<img src="picture 2/p5.png" width="100%">

---

# PHẦN 2: FUNCTION

## Ảnh 6: Bổ sung

- Lệnh SQL:

```sql
SELECT COLUMN_NAME 
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'SanPham';
```

<img src="picture 2/p6.png" width="100%">

---

## Ảnh 7: Built-in Function

- Lệnh SQL:

```sql
USE QL_KhoDienThoai;
GO

IF OBJECT_ID('dbo.DienThoai', 'U') IS NOT NULL
    DROP TABLE dbo.DienThoai;

CREATE TABLE dbo.DienThoai (
    MaDT INT IDENTITY(1,1) PRIMARY KEY,
    TenDT NVARCHAR(100),
    GiaNhap DECIMAL(12,2),
    GiaBan DECIMAL(12,2),
    SoLuong INT
);

INSERT INTO dbo.DienThoai (TenDT, GiaNhap, GiaBan, SoLuong)
VALUES
(N'iPhone 15', 20000000, 25000000, 10),
(N'Samsung S23', 15000000, 18000000, 15),
(N'Redmi Note 12', 4000000, 5000000, 20);

SELECT 
    TenDT,
    LEN(TenDT) AS DoDaiTen,
    UPPER(TenDT) AS TenInHoa,
    GiaBan,
    ROUND(GiaBan, -3) AS GiaLamTron,
    (GiaBan - GiaNhap) AS LoiNhuan,
    GETDATE() AS ThoiGian
FROM dbo.DienThoai;

SELECT 
    COUNT(*) AS SoLuongSP,
    SUM(GiaBan) AS TongGiaBan,
    AVG(GiaBan) AS GiaTrungBinh,
    MAX(GiaBan) AS GiaCaoNhat,
    MIN(GiaBan) AS GiaThapNhat,
    MIN(LEN(TenDT)) AS TenNganNhat,
    MAX(LEN(TenDT)) AS TenDaiNhat,
    GETDATE() AS ThoiGian
FROM dbo.DienThoai;
```

- Mục đích:
Tạo lại bảng đúng chuẩn
Thêm dữ liệu mẫu
Áp dụng đầy đủ built-in function

<img src="picture 2/p7.png" width="100%">

---

## Ảnh 8: SCALAR FUNCTION: Tính tổng tiền đơn hàng

- Lệnh SQL:

```sql
USE QL_KhoDienThoai;
GO

IF OBJECT_ID('dbo.fn_TongTienKho','FN') IS NOT NULL
    DROP FUNCTION dbo.fn_TongTienKho;
GO

CREATE FUNCTION dbo.fn_TongTienKho ()
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE @TongTien DECIMAL(18,2);

    SELECT @TongTien = SUM(SoLuong * GiaBan)
    FROM dbo.DienThoai;

    RETURN ISNULL(@TongTien, 0);
END;
GO

SELECT dbo.fn_TongTienKho() AS TongTienKho;
```

- Mục đích:
Tính tổng tiền của 1 đơn hàng dựa vào bảng ChiTietDonHang
- Kết quả:
Trả về 1 giá trị (tổng tiền) theo MaDH

<img src="picture 2/p8.png" width="100%">

---

## Ảnh 9: Inline Table Function - Lọc sản phẩm sắp hết hàng

- Lệnh SQL:

```sql
USE QL_KhoDienThoai;
GO

INSERT INTO dbo.DienThoai (TenDT, GiaNhap, GiaBan, SoLuong)
VALUES
(N'iPhone 15', 20000000, 25000000, 3),
(N'Samsung S23', 15000000, 18000000, 10),
(N'Redmi Note 12', 4000000, 5000000, 2);

SELECT 
    MaDT,
    TenDT,
    SoLuong,
    GiaBan,
    LEN(TenDT) AS DoDaiTen,
    UPPER(TenDT) AS TenInHoa,
    GETDATE() AS ThoiGian
FROM dbo.DienThoai
WHERE SoLuong <= 5;
```

- Mục đích:
Thêm dữ liệu mẫu
Lọc sản phẩm sắp hết + built-in function
- Kết quả:
Hiển thị bảng gồm:
iPhone 15 (3 cái)
Redmi Note 12 (2 cái)
Có thêm cột xử lý chuỗi + thời gian

<img src="picture 2/p9.png" width="100%">

---

## Ảnh 10: Multi-statement Function - Xếp loại sản phẩm

- Lệnh SQL:

```sql
USE QL_KhoDienThoai;
GO

-- Thêm dữ liệu mẫu (nếu chưa có)
IF NOT EXISTS (SELECT 1 FROM dbo.DienThoai)
BEGIN
    INSERT INTO dbo.DienThoai (TenDT, GiaNhap, GiaBan, SoLuong)
    VALUES
    (N'iPhone 15', 20000000, 25000000, 10),
    (N'Samsung S23', 15000000, 18000000, 5),
    (N'Redmi Note 12', 4000000, 5000000, 2);
END

-- Query xếp loại (logic giống multi-statement function)
SELECT 
    MaDT,
    TenDT,
    SoLuong,
    GiaBan,
    CASE 
        WHEN SoLuong >= 10 THEN N'Còn nhiều'
        WHEN SoLuong BETWEEN 5 AND 9 THEN N'Trung bình'
        ELSE N'Sắp hết'
    END AS XepLoai,
    GETDATE() AS ThoiGian
FROM dbo.DienThoai;
```

- Mục đích:
Mô phỏng logic của Multi-statement Function bằng CASE
Xếp loại sản phẩm theo số lượng
- Kết quả: 
Mỗi sản phẩm có thêm cột XepLoai:
≥10 → Còn nhiều
5–9 → Trung bình
<5 → Sắp hết

<img src="picture 2/p10.png" width="100%">

---

# PHẦN 3: STORED PROCEDURE

## Ảnh 11: SP thêm sản phẩm

- Lệnh SQL:

```sql
USE QL_KhoDienThoai;
GO

IF OBJECT_ID('dbo.sp_ThemSanPham', 'P') IS NOT NULL
    DROP PROCEDURE dbo.sp_ThemSanPham;
GO

CREATE PROCEDURE dbo.sp_ThemSanPham
    @TenDT NVARCHAR(100),
    @GiaNhap DECIMAL(12,2),
    @GiaBan DECIMAL(12,2),
    @SoLuong INT
AS
BEGIN
    INSERT INTO dbo.DienThoai (TenDT, GiaNhap, GiaBan, SoLuong)
    VALUES (@TenDT, @GiaNhap, @GiaBan, @SoLuong);

    SELECT * 
    FROM dbo.DienThoai
    WHERE MaDT = SCOPE_IDENTITY();
END;
```

- Mục đích:
Thêm một sản phẩm mới vào bảng DienThoai
- Kết quả:
Chèn sản phẩm vào database
Trả về sản phẩm vừa thêm

<img src="picture 2/p11.png" width="100%">

---

## Ảnh 12: Gọi SP

- Lệnh SQL:

```sql
EXEC QL_KhoDienThoai.dbo.sp_ThemSanPham 
    N'iPhone 16',
    22000000,
    27000000,
    8;
```
<img src="picture 2/p12.png" width="100%">

---

# PHẦN 4: TRIGGER VÀ XỬ LÝ NGHIỆP VỤ

## Ảnh 13: Trigger tự động trừ kho khi bán hàng

- Lệnh SQL:

```sql
USE QL_KhoDienThoai;
GO

CREATE OR ALTER TRIGGER trg_TruKhoBanHang
ON dbo.ChiTietDonHang
AFTER INSERT
AS
BEGIN
    UPDATE dt
    SET dt.SoLuong = dt.SoLuong - i.SoLuong
    FROM dbo.DienThoai dt
    INNER JOIN inserted i ON dt.MaDT = i.MaSP;
END;
GO
```

- Mục đích:
Khi thêm chi tiết đơn hàng → tự động trừ số lượng trong kho
- Kết quả:
SoLuong trong bảng DienThoai giảm tương ứng số lượng bán

<img src="picture 2/p13.png" width="100%">

---

## Ảnh 14: test Trigger hàng trước khi bán

- Lệnh SQL:

```sql
DECLARE @MaSP INT = 1;
DECLARE @SoLuongBan INT = 2;

-- Kiểm tra tồn kho trước khi bán
SELECT 
    MaDT,
    TenDT,
    SoLuong AS TonKho,
    @SoLuongBan AS SoLuongMuonBan,
    CASE 
        WHEN SoLuong >= @SoLuongBan THEN N'Đủ hàng'
        ELSE N'Không đủ hàng'
    END AS TrangThai
FROM dbo.DienThoai
WHERE MaDT = @MaSP;
```

- Mục đích:
Kiểm tra sản phẩm có đủ số lượng để bán hay không trước khi INSERT vào đơn hàng
- Kết quả:
Hiển thị:
tồn kho hiện tại
số lượng muốn bán
trạng thái “Đủ hàng / Không đủ hàng”

<img src="picture 2/p14.png" width="100%">

---

## Ảnh 15: Trigger chạy thành công

- Lệnh SQL:

```sql
ALTER TABLE dbo.ChiTietDonHang DROP CONSTRAINT FK_CTDH_DH;

ALTER TABLE dbo.ChiTietDonHang
ADD CONSTRAINT FK_CTDH_DH
FOREIGN KEY (MaSP) REFERENCES dbo.DienThoai(MaDT);
```

<img src="picture 2/p15.png" width="100%">

---

## Ảnh 16: Kiểm tra Insert

- Lệnh SQL:

```sql
PRINT N'Tồn kho sau khi bán';

SELECT MaDT, TenDT, SoLuong
FROM dbo.DienThoai
WHERE MaDT = 1;
```

- Ý nghĩa: Kiểm tra insert sau khi trigger chạy thành công

<img src="picture 2/p16.png" width="100%">

## Ảnh 17: Trigger từ bảng A đến B

- Lệnh SQL:

```sql
USE QL_KhoDienThoai;
GO

CREATE OR ALTER TRIGGER trg_A_to_B
ON dbo.ChiTietDonHang
AFTER INSERT
AS
BEGIN
    UPDATE dt
    SET dt.SoLuong = dt.SoLuong - i.SoLuong
    FROM dbo.DienThoai dt
    INNER JOIN inserted i ON dt.MaDT = i.MaSP;
END;
GO
```

- Mục đích: Khi cập nhật sản phẩm sẽ cập nhật đơn hàng

<img src="picture 2/p17.png" width="100%">

---

## Ảnh 18: Trigger từ bảng B đến A

- Lệnh SQL:

```sql
USE QL_KhoDienThoai;
GO

CREATE OR ALTER TRIGGER trg_B_to_A
ON dbo.DienThoai
AFTER UPDATE
AS
BEGIN
    INSERT INTO dbo.ChiTietDonHang (MaDH, MaSP, SoLuong, GiaBan)
    SELECT 
        0 AS MaDH,
        i.MaDT,
        0,
        i.GiaBan
    FROM inserted i
    WHERE i.SoLuong = 0;
END;
GO
```

- Mục đích: Khi bảng B (DienThoai) bị update
Nếu sản phẩm hết hàng (SoLuong = 0)
→ tự động ghi vào A (ChiTietDonHang) như một bản ghi cảnh báo

<img src="picture 2/p18.png" width="100%">

---

# KẾT LUẬN PHẦN TRIGGER

- Trigger là cơ chế tự động thực thi khi có thay đổi dữ liệu (INSERT, UPDATE, DELETE).
- Trong bài kho điện thoại, trigger được dùng để:
Tự động trừ số lượng sản phẩm khi bán hàng (A → B)
Có thể mở rộng để ghi nhận trạng thái hết hàng (B → A)
- Trigger giúp hệ thống:
Tự động hóa thao tác
Giảm lỗi nhập liệu thủ công
Đồng bộ dữ liệu giữa các bảng
- Tuy nhiên cần lưu ý:
Không thiết kế trigger chạy vòng lặp (recursive trigger)
Phải kiểm soát điều kiện để tránh update/insert lặp vô hạn
Cần thiết lập hợp lý giữa các bảng để tránh xung đột dữ liệu

- Kết luận: Trigger là công cụ mạnh trong SQL Server giúp tự động hóa xử lý dữ liệu, nhưng phải thiết kế đúng để đảm bảo hệ thống ổn định và không gây lỗi vòng lặp.

---

# PHẦN 5: CURSOR VÀ DUYỆT DỮ LIỆU

## Ảnh 19: Cursor duyệt sản phẩm trong kho
- Lệnh SQL:

```sql
USE QL_KhoDienThoai;
GO

DECLARE @MaDT INT,
        @TenDT NVARCHAR(100),
        @SoLuong INT;

DECLARE cur_SanPham CURSOR FOR
SELECT MaDT, TenDT, SoLuong
FROM dbo.DienThoai;

OPEN cur_SanPham;

FETCH NEXT FROM cur_SanPham INTO @MaDT, @TenDT, @SoLuong;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT N'Sản phẩm: ' + @TenDT + N' - Số lượng: ' + CAST(@SoLuong AS NVARCHAR);

    FETCH NEXT FROM cur_SanPham INTO @MaDT, @TenDT, @SoLuong;
END;

CLOSE cur_SanPham;
DEALLOCATE cur_SanPham;
```
- Mục đích:
Duyệt từng dòng trong bảng DienThoai
Hiển thị thông tin từng sản phẩm bằng PRINT

<img src="picture 2/p19.png" width="100%">

---

## Ảnh 20: Ngược lại nếu không dùng Cursor 

- Lệnh SQL:

```sql
SELECT 
    MaDT,
    TenDT,
    SoLuong,

    CASE 
        WHEN SoLuong = 0 THEN N'Hết hàng'
        WHEN SoLuong < 5 THEN N'Sắp hết'
        ELSE N'Còn hàng'
    END AS TrangThai,

    COUNT(*) OVER (
        PARTITION BY 
        CASE 
            WHEN SoLuong = 0 THEN N'Hết hàng'
            WHEN SoLuong < 5 THEN N'Sắp hết'
            ELSE N'Còn hàng'
        END
    ) AS TongTheoTrangThai

FROM dbo.DienThoai;
```

- Mục đích:
Thay hoàn toàn Cursor bằng SQL set-based
Vừa:
duyệt dữ liệu
phân loại sản phẩm
- Kết quả:
Mỗi dòng sẽ có:
Thông tin sản phẩm
Trạng thái:
Hết hàng
Sắp hết
Còn hàng

<img src="picture 2/p20.png" width="100%">

---

# SO SÁNH NHANH

|Cursor	            |SQL thuần          |
| ----------------- | ----------------- | 
|Duyệt từng dòng    |Xử lý toàn bộ 1 lần|
|Chậm	            |Nhanh hơn          |
|Dễ viết logic nhỏ	|Dùng CASE, GROUP BY|

---

# NHẬN XÉT:
- Việc sử dụng SQL thuần (set-based) thay cho Cursor giúp xử lý dữ liệu hiệu quả hơn.
Câu lệnh CASE cho phép phân loại sản phẩm theo điều kiện mà không cần duyệt từng dòng.
Hàm COUNT() OVER (PARTITION BY ...) giúp thống kê dữ liệu theo nhóm ngay trong một truy vấn.
- So với Cursor, phương pháp này:
chạy nhanh hơn
tối ưu tài nguyên hơn

---

# KẾT LUẬN:
- SQL thuần là phương pháp tối ưu để xử lý và duyệt dữ liệu trong SQL Server.
Việc thay thế Cursor bằng các câu lệnh như CASE, GROUP BY, WINDOW FUNCTION giúp tăng hiệu suất hệ thống.
- Trong các hệ thống thực tế và bài toán quản lý kho, nên ưu tiên sử dụng SQL set-based thay vì Cursor để đảm bảo tốc độ và hiệu quả xử lý dữ liệu.
