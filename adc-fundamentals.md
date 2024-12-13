# Bài giảng: Nguyên lý và đặc tính cơ bản của ADC

## Mục lục
1. [Giới thiệu về ADC](#1-giới-thiệu-về-adc)
2. [Nguyên lý hoạt động của ADC](#2-nguyên-lý-hoạt-động-của-adc)
3. [Tần số lấy mẫu và độ phân giải](#3-tần-số-lấy-mẫu-và-độ-phân-giải)
4. [Ví dụ thực tế và ứng dụng](#4-ví-dụ-thực-tế-và-ứng-dụng)

## 1. Giới thiệu về ADC

### 1.1. Định nghĩa và vai trò
ADC (Analog to Digital Converter) là thiết bị chuyển đổi tín hiệu từ dạng tương tự (analog) sang dạng số (digital). Đây là cầu nối quan trọng giữa thế giới vật lý (analog) và thế giới số (digital).

### 1.2. Tầm quan trọng trong hệ thống điện tử
- **Xử lý tín hiệu**: Cho phép vi xử lý và máy tính xử lý các tín hiệu từ thế giới thực
- **Đo lường chính xác**: Tạo cơ sở cho các phép đo và điều khiển chính xác
- **Lưu trữ dữ liệu**: Dễ dàng lưu trữ và phân tích thông tin

### 1.3. Ứng dụng thực tế
1. **Y tế**:
   - Thiết bị đo nhịp tim
   - Máy đo huyết áp điện tử
   - Thiết bị siêu âm

2. **Công nghiệp**:
   - Đo nhiệt độ trong lò nung
   - Kiểm soát chất lượng sản phẩm
   - Hệ thống giám sát môi trường

3. **Điện tử tiêu dùng**:
   - Micro trong điện thoại
   - Cảm biến ánh sáng trong máy ảnh
   - Thiết bị đo năng lượng thông minh

## 2. Nguyên lý hoạt động của ADC

### 2.1. Quy trình chuyển đổi cơ bản
1. **Lấy mẫu (Sampling)**:
   - Thu thập giá trị tín hiệu analog tại các thời điểm xác định
   - Tần số lấy mẫu quyết định chất lượng tín hiệu số

2. **Lượng tử hóa (Quantization)**:
   - Chuyển đổi giá trị analog thành các mức rời rạc
   - Số mức phụ thuộc vào độ phân giải

3. **Mã hóa (Encoding)**:
   - Chuyển đổi các mức lượng tử thành mã số nhị phân
   - Tạo ra dữ liệu số cuối cùng

### 2.2. Công thức chuyển đổi
```
N = (Vi * (2^n - 1)) / Vref
```
Trong đó:
- N: Giá trị số đầu ra
- Vi: Điện áp đầu vào analog
- n: Số bit độ phân giải
- Vref: Điện áp tham chiếu

### 2.3. Các thông số quan trọng
1. **Độ phân giải (Resolution)**:
   - Số bit của ADC
   - Quyết định số mức lượng tử

2. **Thời gian chuyển đổi**:
   - Thời gian từ khi bắt đầu đến khi hoàn thành chuyển đổi
   - Ảnh hưởng đến tốc độ lấy mẫu tối đa

3. **Sai số lượng tử**:
   - Sai số do làm tròn giá trị analog
   - Tối đa bằng 1 LSB (Least Significant Bit)

## 3. Tần số lấy mẫu và độ phân giải

### 3.1. Tần số lấy mẫu (Sampling Rate)

#### 3.1.1. Định lý Nyquist-Shannon
- Tần số lấy mẫu phải ít nhất gấp 2 lần tần số cao nhất của tín hiệu
- Fs ≥ 2 * Fmax
- Ngăn ngừa hiện tượng aliasing

#### 3.1.2. Ảnh hưởng của tần số lấy mẫu
1. **Tần số cao**:
   - Ưu điểm:
     * Tái tạo tín hiệu chính xác
     * Giảm mất mát thông tin
   - Nhược điểm:
     * Tốn nhiều bộ nhớ
     * Xử lý phức tạp hơn

2. **Tần số thấp**:
   - Ưu điểm:
     * Tiết kiệm bộ nhớ
     * Xử lý đơn giản
   - Nhược điểm:
     * Mất thông tin
     * Sai lệch tín hiệu

### 3.2. Độ phân giải (Resolution)

#### 3.2.1. Các mức độ phân giải phổ biến
1. **8-bit**:
   - 256 mức (2^8)
   - LSB = Vref/256
   - Ứng dụng: Cảm biến đơn giản

2. **10-bit**:
   - 1024 mức (2^10)
   - LSB = Vref/1024
   - Ứng dụng: Điều khiển công nghiệp

3. **12-bit**:
   - 4096 mức (2^12)
   - LSB = Vref/4096
   - Ứng dụng: Đo lường chính xác

4. **16-bit**:
   - 65536 mức (2^16)
   - LSB = Vref/65536
   - Ứng dụng: Thiết bị y tế, âm thanh chất lượng cao

#### 3.2.2. Ảnh hưởng của độ phân giải
1. **Độ chính xác**:
   - Độ phân giải càng cao càng chính xác
   - Giảm sai số lượng tử

2. **Yêu cầu hệ thống**:
   - Bộ nhớ lớn hơn
   - Thời gian xử lý lâu hơn

## 4. Ví dụ thực tế và ứng dụng

### 4.1. Đo nhiệt độ với LM35
```c
// Ví dụ với ADC 10-bit, Vref = 5V
float temperature = (adc_value * 5.0 / 1024) * 100;
// Độ phân giải nhiệt độ = 5/(1024*0.01) = 0.0488°C
```

### 4.2. Đo ánh sáng với quang trở
```c
// Ví dụ với ADC 8-bit, Vref = 3.3V
float light_level = (adc_value * 3.3) / 256;
// Độ phân giải điện áp = 3.3/256 = 0.0129V
```

### 4.3. Các tính toán thực tế

#### Ví dụ 1: Tính độ phân giải điện áp
```
Với ADC 12-bit, Vref = 5V
Số mức = 2^12 = 4096
LSB = 5V/4096 = 1.22mV
```

#### Ví dụ 2: Tính tần số lấy mẫu cần thiết
```
Tín hiệu âm thanh tần số tối đa 20kHz
Theo Nyquist: Fs ≥ 2 * 20kHz
Fs tối thiểu = 40kHz
Fs thực tế nên chọn ≥ 44.1kHz
```

### 4.4. Các lưu ý quan trọng trong thực tế

1. **Nhiễu và độ ổn định**:
   - Sử dụng tụ bypass
   - Cách ly nguồn analog và digital
   - Thiết kế PCB hợp lý

2. **Tối ưu hóa hệ thống**:
   - Cân bằng giữa độ chính xác và tốc độ
   - Lựa chọn độ phân giải phù hợp
   - Xử lý nhiễu hiệu quả

3. **Hiệu chuẩn và bảo trì**:
   - Hiệu chuẩn định kỳ
   - Kiểm tra độ trôi
   - Bảo trì phòng ngừa

---
© 2024 Embedded Systems Education. Tài liệu này được tạo cho mục đích giáo dục.
