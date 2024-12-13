# Bài giảng: Nguyên lý quét LED và phân tích chi tiết

## I. Nguyên lý cơ bản quét LED

### 1.1. Cơ chế hoạt động
1. **Tính chất tồn lưu của mắt**
   - Mắt người giữ lại hình ảnh khoảng 1/16 giây
   - Nếu quét LED nhanh hơn 16ms, sẽ tạo ảo giác sáng liên tục

2. **Nguyên lý làm việc**
   ```plaintext
   Thời gian |  D3   |  D2   |  D1   |  D0   |
   0-3ms     |  ON   |  OFF  |  OFF  |  OFF  |
   3-6ms     |  OFF  |  ON   |  OFF  |  OFF  |
   6-9ms     |  OFF  |  OFF  |  ON   |  OFF  |
   9-12ms    |  OFF  |  OFF  |  OFF  |  ON   |
   ```

3. **Yêu cầu tần số**
   - Tần số quét = 1/(số digit × thời gian mỗi digit)
   - Ví dụ: 4 digit × 3ms = 12ms/chu kỳ
   - Tần số ≈ 83Hz > 60Hz (ngưỡng nhấp nháy)

### 1.2. Cấu trúc phần cứng
1. **Kết nối LED 7 đoạn**
   ```plaintext
   PORT B (RB0-RB7): Điều khiển các đoạn LED
   PORT D (RD0-RD3): Điều khiển các digit
   
   Digit select: Active LOW
   Segment drive: Active HIGH
   ```

2. **Mạch điều khiển**
   ```plaintext
   LED 7 đoạn → Điện trở hạn dòng → PORT B
   Digit select → Transistor → PORT D
   ```

## II. Phân tích chi tiết code

### 2.1. Cấu hình hệ thống
```c
#include <16F877A.h>
#device ADC=10             // ADC 10-bit
#use delay(clock=8000000)  // Tần số 8MHz

// Mã LED 7 đoạn
CONST INT8 a[10] = {
    0x03,  // 0: abcdef-   → 0000 0011
    0x9f,  // 1: -bc----   → 1001 1111
    0x25,  // 2: ab-de-g   → 0010 0101
    0x0d,  // 3: abcd--g   → 0000 1101
    ...
};

// Biến điều khiển
int8 chuc, dvi;  // Lưu chữ số hàng chục, đơn vị
int16 temp;      // Giá trị nhiệt độ
```

### 2.2. Cấu trúc Timer0 cho quét LED
```c
// Timer0 setup
Setup_timer_0(RTCC_DIV_4|RTCC_INTERNAL);
/*
Với thạch anh 8MHz:
Chu kỳ cơ bản = 1/8MHz = 0.125µs
Bộ chia 4 → 0.5µs
Timer đếm từ 200 đến tràn (255)
Thời gian ngắt = (256-200) × 0.5µs = 28µs
*/

// Cho phép ngắt
enable_interrupts(GLOBAL);
enable_interrupts(INT_TIMER0);
set_timer0(200);  // Nạp giá trị khởi tạo
```

### 2.3. Chi tiết hàm quét LED
```c
#INT_TIMER0
void qled() {
    static uint8_t step = 0;  // Biến theo dõi bước quét
    
    switch(step) {
        case 0:  // Quét số hàng chục
            Output_d(0xff);         // Tắt tất cả
            Output_b(a[chuc]);      // Xuất mã LED
            output_low(PIN_D3);     // Bật digit
            step++;
            break;
            
        case 1:  // Quét số hàng đơn vị
            Output_d(0xff);
            Output_b(a[dvi]);
            output_low(PIN_D2);
            step++;
            break;
            
        case 2:  // Quét dấu độ
            Output_d(0xff);
            Output_b(0x39);         // Mã hiển thị °
            output_low(PIN_D1);
            step++;
            break;
            
        case 3:  // Quét chữ C
            Output_d(0xff);
            Output_b(0x63);         // Mã hiển thị C
            output_low(PIN_D0);
            step = 0;
            break;
    }
    
    delay_ms(3);  // Duy trì hiển thị 3ms
}
```

## III. Giải thích từng bước quét LED

### 3.1. Quy trình quét cơ bản
1. **Bước 1: Tắt tất cả LED**
   ```c
   Output_d(0xff);  // Tất cả digit OFF
   ```
   - Tránh hiện tượng chồng chéo
   - Đảm bảo hiển thị sạch

2. **Bước 2: Chuẩn bị dữ liệu**
   ```c
   Output_b(a[chuc]);  // Đưa ra mã LED
   ```
   - Lấy mã từ mảng lookup
   - Đưa ra PORT B

3. **Bước 3: Kích hoạt digit**
   ```c
   output_low(PIN_D3);  // Bật digit cụ thể
   ```
   - Chỉ một digit active tại một thời điểm
   - Active LOW (0 = ON, 1 = OFF)

4. **Bước 4: Duy trì hiển thị**
   ```c
   delay_ms(3);  // Đợi 3ms
   ```
   - Thời gian đủ để mắt nhận biết
   - Không quá dài để tránh nhấp nháy

### 3.2. Lưu ý quan trọng
1. **Thời gian quét**
   - Tổng thời gian một chu kỳ = 12ms (4 × 3ms)
   - Tần số quét = 83.33Hz
   - Đủ nhanh để tránh nhấp nháy

2. **Độ sáng**
   - Phụ thuộc vào duty cycle
   - 3ms ON / 12ms chu kỳ = 25% duty cycle

3. **Hiện tượng chồng chéo**
   - Tắt tất cả trước khi bật digit mới
   - Đảm bảo hiển thị rõ ràng

## IV. Ví dụ thực tế

### 4.1. Hiển thị nhiệt độ 29°C
```plaintext
Bước 1: chuc = 2    // Hiển thị số 2
        PIN_D3 = 0
        PORT_B = 0x25
        Delay 3ms

Bước 2: dvi = 9     // Hiển thị số 9
        PIN_D2 = 0
        PORT_B = 0x09
        Delay 3ms

Bước 3: Dấu độ     // Hiển thị °
        PIN_D1 = 0
        PORT_B = 0x39
        Delay 3ms

Bước 4: Chữ C      // Hiển thị C
        PIN_D0 = 0
        PORT_B = 0x63
        Delay 3ms
```

### 4.2. Debug và tối ưu
```c
// Thêm biến debug
volatile uint8_t current_digit = 0;
volatile uint8_t display_data = 0;

// Trong hàm ngắt
void qled() {
    // Lưu thông tin debug
    current_digit = step;
    display_data = PORT_B;
    
    // Tiếp tục quét LED như bình thường
    ...
}
```

---
© 2024 Embedded Systems Education. Tài liệu này được tạo cho mục đích giáo dục.