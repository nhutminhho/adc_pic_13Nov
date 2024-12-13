# Bài giảng: Phân tích code đo nhiệt độ LM35 và hiển thị LCD

## I. Phân tích cấu trúc code

### 1.1. Cấu hình ban đầu
```c
#include <16F877A.h>            // Thư viện cho PIC16F877A
#device adc=10                  // Cấu hình ADC 10-bit
#FUSES NOWDT, HS, NOPUT, PROTECT, NODEBUG, NOBROWNOUT, NOLVP
#use delay(clock=8000000)       // Cấu hình tần số dao động 8MHz
#include <lcd.c>                // Thư viện LCD
```

#### Giải thích các cấu hình Fuse:
- **NOWDT**: Tắt Watch Dog Timer
- **HS**: Sử dụng dao động thạch anh tốc độ cao
- **NOPUT**: Không sử dụng Power Up Timer
- **PROTECT**: Bảo vệ code
- **NODEBUG**: Không sử dụng chế độ debug
- **NOBROWNOUT**: Tắt chức năng phát hiện điện áp thấp
- **NOLVP**: Tắt chế độ lập trình điện áp thấp

### 1.2. Hàm main
```c
void main() {
    INT temp;                           // Biến lưu nhiệt độ
    Setup_adc(ADC_CLOCK_INTERNAL);      // Cấu hình xung clock ADC
    Setup_adc_ports(AN0);               // Cấu hình port AN0 làm input
    Set_adc_channel(0);                 // Chọn kênh AD0
```

#### Giải thích cấu hình ADC:
1. **Setup_adc(ADC_CLOCK_INTERNAL)**
   - Sử dụng nguồn xung nội cho ADC
   - Phù hợp với tần số hoạt động của vi điều khiển

2. **Setup_adc_ports(AN0)**
   - Chỉ cấu hình AN0 làm đầu vào analog
   - Các chân khác vẫn là digital I/O

3. **Set_adc_channel(0)**
   - Chọn kênh 0 (AN0) để đọc giá trị
   - Kết nối với đầu ra của LM35

### 1.3. Khởi tạo LCD và hiển thị tiêu đề
```c
lcd_init();                             // Khởi tạo LCD
temp = 0;                               // Khởi tạo biến temp
lcd_gotoxy(1, 1);                       // Di chuyển con trỏ đến cột 1, hàng 1
printf(lcd_putc, "Nhiet do phong:");    // In tiêu đề
```

#### Giải thích các hàm LCD:
1. **lcd_init()**
   - Khởi tạo LCD ở chế độ 4-bit
   - Xóa màn hình
   - Cấu hình các thông số cơ bản

2. **lcd_gotoxy(x, y)**
   - Di chuyển con trỏ đến vị trí cần hiển thị
   - x: vị trí cột (1-16)
   - y: vị trí hàng (1-2)

### 1.4. Vòng lặp chính
```c
WHILE (TRUE) {
    temp = read_adc() / 2.046;          // Đọc và chuyển đổi giá trị nhiệt độ
    lcd_gotoxy(6, 2);                   // Di chuyển đến vị trí hiển thị nhiệt độ
    printf(lcd_putc, " %02u do C", temp); // In giá trị nhiệt độ
}
```

#### Giải thích công thức chuyển đổi:
1. **read_adc() / 2.046**
   - read_adc(): Đọc giá trị ADC (0-1023)
   - 2.046: Hệ số chuyển đổi được tính từ:
     * Độ nhạy LM35: 10mV/°C
     * Độ phân giải ADC: 4.887mV/bit
     * Tỷ số: 10/4.887 = 2.046

2. **Format hiển thị**
   - %02u: Hiển thị số nguyên không dấu
   - Chiếm 2 vị trí, thêm số 0 phía trước nếu cần
   - "do C": Đơn vị nhiệt độ

## II. Các cải tiến có thể thực hiện

### 2.1. Thêm lọc nhiễu
```c
int read_temp_filtered() {
    int32 sum = 0;
    for(int i=0; i<8; i++) {
        sum += read_adc();
        delay_ms(1);
    }
    return sum / (8 * 2.046);
}
```

### 2.2. Thêm hiển thị số thập phân
```c
float temp;
temp = read_adc() / 2.046;
printf(lcd_putc, " %3.1f do C", temp);
```

### 2.3. Thêm cảnh báo nhiệt độ
```c
if(temp > 35) {
    output_high(PIN_B0);    // Bật đèn LED cảnh báo
    lcd_gotoxy(15, 2);
    lcd_putc("!");          // Hiển thị ký hiệu cảnh báo
}
```

## III. Debugging và xử lý lỗi

### 3.1. Các lỗi thường gặp
1. **Giá trị nhiệt độ không chính xác**
   - Kiểm tra hệ số chuyển đổi
   - Kiểm tra điện áp tham chiếu
   - Xác nhận kết nối phần cứng

2. **LCD không hiển thị**
   - Kiểm tra kết nối LCD
   - Xác nhận khởi tạo LCD
   - Kiểm tra độ tương phản LCD

3. **Đọc ADC không ổn định**
   - Thêm tụ lọc nhiễu
   - Tăng thời gian trễ giữa các lần đọc
   - Sử dụng kỹ thuật lọc trung bình

## IV. Bài tập thực hành

1. **Cơ bản**:
   - Thêm hiển thị nhiệt độ theo °F
   - Lưu giá trị max/min

2. **Nâng cao**:
   - Thêm chức năng cảnh báo qua buzzer
   - Hiển thị đồ thị nhiệt độ đơn giản
   - Lưu dữ liệu vào EEPROM

---
© 2024 Embedded Systems Education. Tài liệu này được tạo cho mục đích giáo dục.