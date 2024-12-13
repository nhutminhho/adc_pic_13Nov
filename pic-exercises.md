# 5 Bài tập PIC đơn giản với ADC và LED

## Bài 1: Hiển thị số từ 0-9 trên LED 7 đoạn

### Yêu cầu
- Hiển thị số từ 0-9 trên LED 7 đoạn đơn
- Mỗi số hiển thị trong 1 giây
- Lặp lại liên tục

### Code mẫu
```c
#include <16F877A.h>
#fuses NOWDT, HS, NOPUT, NOPROTECT
#use delay(clock=8M)

// Mã LED 7 đoạn chung cực dương
int8 LED7S[10] = {0x03, 0x9f, 0x25, 0x0d, 0x99, 0x49, 0x41, 0x1f, 0x01, 0x09};

void main() {
    while(1) {
        for(int8 i = 0; i < 10; i++) {
            output_b(LED7S[i]);  // Xuất ra PORTB
            delay_ms(1000);      // Đợi 1 giây
        }
    }
}
```

## Bài 2: Đọc ADC và hiển thị LED đơn

### Yêu cầu
- Đọc giá trị từ ADC kênh AN0
- Nếu giá trị > 512 thì bật LED ở RC0
- Nếu giá trị ≤ 512 thì tắt LED

### Code mẫu
```c
#include <16F877A.h>
#fuses NOWDT, HS, NOPUT, NOPROTECT
#use delay(clock=8M)

void main() {
    int16 adc_value;
    
    setup_adc(ADC_CLOCK_INTERNAL);
    setup_adc_ports(AN0);
    set_adc_channel(0);
    
    while(1) {
        adc_value = read_adc();
        if(adc_value > 512)
            output_high(PIN_C0);
        else
            output_low(PIN_C0);
        delay_ms(100);
    }
}
```

## Bài 3: Hiển thị LED theo mức điện áp

### Yêu cầu
- Đọc điện áp từ AN0 (0-5V)
- Hiển thị số LED sáng tương ứng với mức điện áp:
  - 0-1V: 1 LED
  - 1-2V: 2 LED
  - 2-3V: 3 LED
  - 3-4V: 4 LED
  - 4-5V: 5 LED

### Code mẫu
```c
#include <16F877A.h>
#fuses NOWDT, HS, NOPUT, NOPROTECT
#use delay(clock=8M)

void main() {
    int16 adc_value;
    float voltage;
    
    setup_adc(ADC_CLOCK_INTERNAL);
    setup_adc_ports(AN0);
    set_adc_channel(0);
    
    while(1) {
        adc_value = read_adc();
        voltage = (float)adc_value * 5 / 1023; // Chuyển đổi sang điện áp
        
        // Tắt hết LED
        output_b(0x00);
        
        // Bật LED theo mức điện áp
        if(voltage > 0) output_high(PIN_B0);
        if(voltage > 1) output_high(PIN_B1);
        if(voltage > 2) output_high(PIN_B2);
        if(voltage > 3) output_high(PIN_B3);
        if(voltage > 4) output_high(PIN_B4);
        
        delay_ms(100);
    }
}
```

## Bài 4: Hiển thị một số trên LED 7 đoạn

### Yêu cầu
- Đọc giá trị ADC và chuyển thành số từ 0-9
- Hiển thị số đó trên LED 7 đoạn
- Cập nhật mỗi 500ms

### Code mẫu
```c
#include <16F877A.h>
#fuses NOWDT, HS, NOPUT, NOPROTECT
#use delay(clock=8M)

int8 LED7S[10] = {0x03, 0x9f, 0x25, 0x0d, 0x99, 0x49, 0x41, 0x1f, 0x01, 0x09};

void main() {
    int16 adc_value;
    int8 number;
    
    setup_adc(ADC_CLOCK_INTERNAL);
    setup_adc_ports(AN0);
    set_adc_channel(0);
    
    while(1) {
        adc_value = read_adc();
        number = adc_value * 10 / 1024;  // Chuyển đổi sang số 0-9
        output_b(LED7S[number]);         // Hiển thị số
        delay_ms(500);
    }
}
```

## Bài 5: Điều khiển LED và còi báo

### Yêu cầu
- Đọc điện áp từ AN0
- Nếu điện áp < 1V: Tắt hết
- Nếu điện áp 1-3V: Bật LED xanh (RC0)
- Nếu điện áp 3-4V: Bật LED đỏ (RC1)
- Nếu điện áp > 4V: Bật còi báo (RC2) và nhấp nháy LED đỏ

### Code mẫu
```c
#include <16F877A.h>
#fuses NOWDT, HS, NOPUT, NOPROTECT
#use delay(clock=8M)

void main() {
    int16 adc_value;
    float voltage;
    
    setup_adc(ADC_CLOCK_INTERNAL);
    setup_adc_ports(AN0);
    set_adc_channel(0);
    
    while(1) {
        adc_value = read_adc();
        voltage = (float)adc_value * 5 / 1023;
        
        // Tắt tất cả
        output_low(PIN_C0);
        output_low(PIN_C1);
        output_low(PIN_C2);
        
        if(voltage >= 1 && voltage < 3) {
            output_high(PIN_C0);  // LED xanh
        }
        else if(voltage >= 3 && voltage < 4) {
            output_high(PIN_C1);  // LED đỏ
        }
        else if(voltage >= 4) {
            output_high(PIN_C2);  // Còi báo
            output_toggle(PIN_C1); // LED đỏ nhấp nháy
        }
        
        delay_ms(100);
    }
}
```

## Giải thích các điểm quan trọng

### 1. Cấu hình cơ bản
- Sử dụng PIC16F877A
- Tần số 8MHz
- Không dùng watchdog timer

### 2. ADC
- Sử dụng ADC nội với kênh AN0
- Độ phân giải 10 bit (0-1023)
- Công thức chuyển đổi sang điện áp: V = ADC * 5 / 1023

### 3. LED 7 đoạn
- Sử dụng mảng để lưu mã hiển thị
- LED chung cực dương (0 = sáng, 1 = tắt)
- Xuất dữ liệu qua PORTB

### 4. Điều khiển ngõ ra
- Sử dụng các hàm output_high() và output_low()
- Có thể dùng output_toggle() để nhấp nháy
- Delay để tạo thời gian trễ

### 5. Cấu trúc chương trình
- Khởi tạo các thiết bị ngoại vi
- Vòng lặp vô tận while(1)
- Đọc ADC và xử lý
- Xuất kết quả ra LED/còi báo

## Phát triển thêm
1. Thêm nút nhấn để chọn chế độ hiển thị
2. Sử dụng Timer để tạo thời gian chính xác
3. Thêm chức năng lưu giá trị max/min
4. Hiển thị nhiều chữ số
5. Thêm giao tiếp UART để gửi dữ liệu về máy tính