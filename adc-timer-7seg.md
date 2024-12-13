# Bài giảng: Đo nhiệt độ và hiển thị LED 7 đoạn sử dụng ngắt Timer0

## I. Tổng quan chương trình

### 1.1. Các thành phần chính
1. **Cấu hình cơ bản**:
   - Vi điều khiển PIC16F877A
   - ADC 10-bit
   - Ngắt Timer0
   - LED 7 đoạn 4 số

2. **Chức năng**:
   - Đo nhiệt độ qua ADC
   - Hiển thị trên LED 7 đoạn
   - Quét LED động sử dụng ngắt Timer0

## II. Phân tích code chi tiết

### 2.1. Cấu hình ban đầu
```c
#include <16F877A.h>
#device ADC=10                  // ADC 10-bit
#FUSES NOWDT,HS,NOPUT,PROTECT,NODEBUG,NOBROWNOUT,NOLVP
#use delay(clock=8000000)       // Tần số 8MHz
```

#### Giải thích các thiết lập Fuse:
- NOWDT: Tắt Watchdog Timer
- HS: Sử dụng dao động thạch anh tốc độ cao
- NOPUT: Không sử dụng Power Up Timer
- Các thiết lập bảo vệ và debug

### 2.2. Định nghĩa mảng mã LED
```c
CONST INT8 a[10] = {0x03,0x9f,0x25,0x0d,0x99,0x49,0x41,0x1f,0x01,0x09};
```
- Mảng chứa mã hiển thị số 0-9 trên LED 7 đoạn
- Mỗi byte đại diện cho các đoạn LED cần sáng

### 2.3. Hàm ngắt Timer0
```c
#INT_TIMER0
void qled() {
    Output_d(0xff);                    // Tắt tất cả LED
    Output_b(a[chuc]); output_low(PIN_D3); delay_ms(3);  // Hiển thị chữ số hàng chục
    Output_d(0xff);                    // Tắt LED
    Output_b(a[dvi]); output_low(PIN_D2); delay_ms(3);   // Hiển thị chữ số hàng đơn vị
    Output_d(0xff);                    // Tắt LED
    Output_b(0x39); output_low(PIN_D1); delay_ms(3);     // Hiển thị dấu độ
    Output_d(0xff);                    // Tắt LED
    Output_b(0x63); output_low(PIN_D0); delay_ms(3);     // Hiển thị chữ C
}
```

#### Nguyên lý quét LED:
1. **Quét tuần tự**:
   - Tắt tất cả LED trước mỗi lần hiển thị
   - Hiển thị từng chữ số và ký tự lần lượt
   - Delay 3ms giữa các lần quét

2. **Thứ tự hiển thị**:
   - Chữ số hàng chục (PIN_D3)
   - Chữ số hàng đơn vị (PIN_D2)
   - Dấu độ '°' (PIN_D1)
   - Chữ 'C' (PIN_D0)

### 2.4. Chương trình chính
```c
void main() {
    // Cấu hình ngắt Timer0
    enable_interrupts(GLOBAL);
    enable_interrupts(INT_TIMER0);
    Setup_timer_0(RTCC_DIV_4|RTCC_INTERNAL);
    set_timer0(200);

    // Cấu hình ADC
    setup_adc(ADC_CLOCK_INTERNAL);
    setup_adc_ports(AN0);
    set_adc_channel(0);

    while(1) {
        set_timer0(200);
        temp = read_adc() / 2.046;     // Đọc và chuyển đổi nhiệt độ
        chuc = temp / 10;              // Tách số hàng chục
        dvi = temp % 10;               // Tách số hàng đơn vị
    }
}
```

#### Giải thích các cấu hình:
1. **Timer0**:
   - Sử dụng bộ chia 4 (RTCC_DIV_4)
   - Nguồn xung nội (RTCC_INTERNAL)
   - Giá trị nạp lại: 200

2. **ADC**:
   - Xung ADC nội (ADC_CLOCK_INTERNAL)
   - Sử dụng kênh AN0
   - Hệ số chuyển đổi: 2.046

3. **Xử lý nhiệt độ**:
   - Đọc giá trị ADC và chuyển đổi
   - Tách thành chữ số hàng chục và đơn vị

## III. Tối ưu và cải tiến

### 3.1. Cải thiện độ ổn định
```c
float read_temp_average() {
    int32 sum = 0;
    for(int i=0; i<8; i++) {
        sum += read_adc();
        delay_ms(1);
    }
    return sum / (8 * 2.046);
}
```

### 3.2. Thêm tính năng
```c
// Thêm cảnh báo nhiệt độ cao
if(temp > 50) {
    output_high(PIN_C0);    // Bật LED cảnh báo
} else {
    output_low(PIN_C0);
}
```

## IV. Bài tập thực hành

1. **Cơ bản**:
   - Thêm hiển thị dấu thập phân
   - Chuyển đổi sang độ F

2. **Nâng cao**:
   - Lưu nhiệt độ max/min
   - Thêm chức năng cảnh báo
   - Hiển thị biểu đồ cột đơn giản

## V. Debug và xử lý lỗi

1. **Các lỗi thường gặp**:
   - Nhiễu ADC
   - LED nhấp nháy
   - Hiển thị không đúng số

2. **Giải pháp**:
   - Thêm tụ lọc nhiễu
   - Điều chỉnh thời gian quét
   - Kiểm tra mã LED 7 đoạn

---
© 2024 Embedded Systems Education. Tài liệu này được tạo cho mục đích giáo dục.