# Bài giảng: Kỹ thuật quét LED động và phân tích chương trình đo nhiệt độ

## I. Tổng quan về kỹ thuật quét LED

### 1.1. Nguyên lý cơ bản
1. **Quét LED là gì?**
   - Là kỹ thuật bật/tắt tuần tự các LED
   - Tốc độ quét nhanh tạo ảo giác sáng đồng thời
   - Tiết kiệm chân I/O và năng lượng

2. **Ưu điểm của quét LED**:
   - Giảm số chân điều khiển
   - Giảm công suất tiêu thụ
   - Dễ mở rộng số lượng LED

### 1.2. Yêu cầu của hệ thống
1. **Tần số quét**:
   - Tối thiểu 50Hz để tránh nhấp nháy
   - Thông thường 100-200Hz
   - Công thức: f = 1/(n × t)
     * n: số digit
     * t: thời gian hiển thị mỗi digit

2. **Thời gian**:
   - Thời gian quét mỗi digit: 3-5ms
   - Tổng thời gian một chu kỳ: 12-20ms
   - Độ trễ giữa các lần quét: 100-200µs

## II. Phân tích chi tiết chương trình

### 2.1. Cấu trúc dữ liệu
```c
// Mảng mã LED 7 đoạn
CONST INT8 a[10] = {
    0x03,  // Số 0: 0000 0011
    0x9f,  // Số 1: 1001 1111
    0x25,  // Số 2: 0010 0101
    0x0d,  // Số 3: 0000 1101
    0x99,  // Số 4: 1001 1001
    0x49,  // Số 5: 0100 1001
    0x41,  // Số 6: 0100 0001
    0x1f,  // Số 7: 0001 1111
    0x01,  // Số 8: 0000 0001
    0x09   // Số 9: 0000 1001
};

// Biến lưu chữ số
int8 chuc, dvi;   // Biến hàng chục và đơn vị
int16 temp;       // Giá trị nhiệt độ
```

### 2.2. Cấu hình Timer0 cho quét LED
```c
// Thiết lập Timer0
enable_interrupts(GLOBAL);        // Cho phép ngắt toàn cục
enable_interrupts(INT_TIMER0);    // Cho phép ngắt Timer0
Setup_timer_0(RTCC_DIV_4|RTCC_INTERNAL);  // Xung nội, chia 4
set_timer0(200);                 // Giá trị nạp lại

// Tính toán thời gian ngắt:
// Với thạch anh 8MHz, bộ chia 4:
// T = (256-200) × 4 × (1/8MHz) = 28µs
```

### 2.3. Hàm quét LED chi tiết
```c
#INT_TIMER0
void qled() {
    // Quét LED số hàng chục
    Output_d(0xff);              // Tắt tất cả LED
    Output_b(a[chuc]);          // Đưa ra mã LED
    output_low(PIN_D3);         // Bật LED hàng chục
    delay_ms(3);                // Duy trì 3ms
    
    // Quét LED số hàng đơn vị
    Output_d(0xff);              // Tắt tất cả LED
    Output_b(a[dvi]);           // Đưa ra mã LED
    output_low(PIN_D2);         // Bật LED hàng đơn vị
    delay_ms(3);                // Duy trì 3ms
    
    // Quét LED dấu độ
    Output_d(0xff);              // Tắt tất cả LED
    Output_b(0x39);             // Mã hiển thị °
    output_low(PIN_D1);         // Bật LED dấu độ
    delay_ms(3);                // Duy trì 3ms
    
    // Quét LED chữ C
    Output_d(0xff);              // Tắt tất cả LED
    Output_b(0x63);             // Mã hiển thị C
    output_low(PIN_D0);         // Bật LED chữ C
    delay_ms(3);                // Duy trì 3ms
}
```

## III. Giải thích chi tiết quy trình quét LED

### 3.1. Quy trình quét từng bước
1. **Bước 1: Tắt tất cả LED**
   - Output_d(0xff) tắt tất cả digit
   - Tránh hiện tượng chồng chéo

2. **Bước 2: Chuẩn bị dữ liệu**
   - Output_b(data) đưa ra mã LED
   - Data lấy từ mảng a[] hoặc mã cố định

3. **Bước 3: Bật digit**
   - output_low() kích hoạt từng digit
   - Thứ tự: D3->D2->D1->D0

4. **Bước 4: Duy trì hiển thị**
   - delay_ms(3) giữ hiển thị
   - Thời gian đủ để mắt nhận biết

### 3.2. Xử lý dữ liệu hiển thị
```c
// Trong vòng lặp chính
while(1) {
    // Đọc ADC và chuyển đổi nhiệt độ
    temp = read_adc() / 2.046;
    
    // Tách số thành các chữ số
    chuc = temp / 10;    // Lấy hàng chục
    dvi = temp % 10;     // Lấy hàng đơn vị
}
```

## IV. Tối ưu hiệu năng

### 4.1. Tối ưu thời gian quét
```c
// Cải thiện hàm quét để giảm thời gian trễ
void qled_optimized() {
    static uint8_t digit = 0;
    Output_d(0xff);    // Tắt tất cả
    
    switch(digit) {
        case 0:
            Output_b(a[chuc]);
            output_low(PIN_D3);
            break;
        case 1:
            Output_b(a[dvi]);
            output_low(PIN_D2);
            break;
        case 2:
            Output_b(0x39);
            output_low(PIN_D1);
            break;
        case 3:
            Output_b(0x63);
            output_low(PIN_D0);
            break;
    }
    
    delay_ms(2);       // Giảm thời gian delay
    digit = (digit + 1) % 4;
}
```

### 4.2. Xử lý nhấp nháy
```c
// Thêm bộ lọc cho giá trị nhiệt độ
int16 temp_old = 0;
while(1) {
    int16 temp_new = read_adc() / 2.046;
    
    // Chỉ cập nhật khi thay đổi đáng kể
    if(abs(temp_new - temp_old) > 1) {
        temp = temp_new;
        temp_old = temp_new;
        
        chuc = temp / 10;
        dvi = temp % 10;
    }
}
```

## V. Bài tập thực hành

### 5.1. Bài tập cơ bản
1. Thêm hiệu ứng sáng dần/tối dần
2. Hiển thị số âm
3. Thêm chức năng nhấp nháy khi cảnh báo

### 5.2. Bài tập nâng cao
1. Tạo hiệu ứng chạy chữ
2. Điều chỉnh độ sáng tự động theo ánh sáng môi trường
3. Tối ưu hóa mã nguồn để giảm thời gian xử lý

---
© 2024 Embedded Systems Education. Tài liệu này được tạo cho mục đích giáo dục.