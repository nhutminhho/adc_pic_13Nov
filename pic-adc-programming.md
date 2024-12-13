# Bài giảng: Lập trình ADC với PIC16F877A

## Mục lục
1. [Cấu trúc ADC trong PIC16F877A](#1-cấu-trúc-adc-trong-pic16f877a)
2. [Các thanh ghi ADC](#2-các-thanh-ghi-adc)
3. [Lập trình ADC với CCS](#3-lập-trình-adc-với-ccs)
4. [Ví dụ thực hành](#4-ví-dụ-thực-hành)

## 1. Cấu trúc ADC trong PIC16F877A

### 1.1. Đặc điểm chính
- ADC 10-bit tích hợp
- 8 kênh đầu vào analog (AN0-AN7)
- Thời gian chuyển đổi: 19.2µs (với Fosc = 20MHz)
- Điện áp tham chiếu linh hoạt (nội/ngoại)

### 1.2. Các khối chức năng chính

#### 1.2.1. Đầu vào Analog
- **AN0/RA0**: Kênh analog 0
- **AN1/RA1**: Kênh analog 1
- **AN2/RA2**: Kênh analog 2/Vref-
- **AN3/RA3**: Kênh analog 3/Vref+
- **AN4/RA5**: Kênh analog 4
- **AN5/RE0**: Kênh analog 5
- **AN6/RE1**: Kênh analog 6
- **AN7/RE2**: Kênh analog 7

#### 1.2.2. Điện áp tham chiếu
1. **VREF+**: 
   - Có thể chọn VDD
   - Hoặc điện áp ngoài qua chân RA3
   - Phải lớn hơn điện áp đầu vào

2. **VREF-**:
   - Có thể chọn VSS
   - Hoặc điện áp ngoài qua chân RA2
   - Thường được nối mass

#### 1.2.3. Bộ Multiplexer
- Chọn một trong 8 kênh analog
- Điều khiển bởi bits CHS2:CHS0
- Cho phép quét nhiều kênh

## 2. Các thanh ghi ADC

### 2.1. ADCON0 (Thanh ghi điều khiển 0)
```
bit 7-6: ADCS1:ADCS0   - Chọn tần số chuyển đổi
bit 5-3: CHS2:CHS0     - Chọn kênh analog
bit 2: GO/DONE         - Trạng thái chuyển đổi
bit 1: ADON           - Bật/tắt module ADC
bit 0: Không sử dụng
```

### 2.2. ADCON1 (Thanh ghi điều khiển 1)
```
bit 7: ADFM           - Định dạng kết quả
bit 6: Không sử dụng
bit 5-0: PCFG5:PCFG0  - Cấu hình cổng analog
```

### 2.3. ADRESH và ADRESL
- Chứa kết quả chuyển đổi 10-bit
- Có thể căn phải hoặc căn trái (ADFM)

## 3. Lập trình ADC với CCS

### 3.1. Các hàm cơ bản

#### 3.1.1. Cấu hình ADC
```c
// Cấu hình độ phân giải ADC
#DEVICE ADC=10   // ADC 10-bit

// Cấu hình xung nhịp ADC
setup_adc(ADC_CLOCK_INTERNAL);  // Sử dụng xung nội
setup_adc(ADC_CLOCK_DIV_32);    // Chia tần số cho 32

// Cấu hình cổng ADC
setup_adc_ports(ALL_ANALOG);     // Tất cả là cổng analog
setup_adc_ports(AN0_AN1_VSS_VREF); // Chỉ AN0, AN1 là analog
```

#### 3.1.2. Đọc giá trị ADC
```c
// Chọn kênh
set_adc_channel(0);  // Chọn kênh AN0
delay_us(10);        // Đợi ổn định

// Đọc giá trị
value = read_adc();  // Đọc giá trị ADC
```

### 3.2. Các chế độ cấu hình cổng

1. **ALL_ANALOG**:
   - Tất cả các chân có thể làm ADC
   - A0, A1, A2, A3, A5, E0, E1, E2
   ```c
   setup_adc_ports(ALL_ANALOG);
   ```

2. **AN0_AN1_AN2_AN4_AN5_AN6_AN7_VSS_VREF**:
   - 7 kênh analog
   - VSS làm điện áp tham chiếu
   ```c
   setup_adc_ports(AN0_AN1_AN2_AN4_AN5_AN6_AN7_VSS_VREF);
   ```

3. **AN0_AN1_AN2_AN3_AN4**:
   - 5 kênh analog đầu
   ```c
   setup_adc_ports(AN0_AN1_AN2_AN3_AN4);
   ```

4. **AN0_VREF_VREF**:
   - Chỉ AN0 là analog
   - A3 làm VREF+
   - A2 làm VREF-
   ```c
   setup_adc_ports(AN0_VREF_VREF);
   ```

## 4. Ví dụ thực hành

### 4.1. Đo nhiệt độ với LM35
```c
#include <16F877A.h>
#device ADC=10
#use delay(crystal=20000000)
#use rs232(baud=9600, xmit=PIN_C6, rcv=PIN_C7)

void main() {
   float temperature;
   int16 adc_value;
   
   // Cấu hình ADC
   setup_adc_ports(AN0_AN1_AN2_AN3_AN4);
   setup_adc(ADC_CLOCK_INTERNAL);
   
   while(1) {
      // Đọc giá trị từ LM35 ở kênh AN0
      set_adc_channel(0);
      delay_us(10);
      adc_value = read_adc();
      
      // Chuyển đổi sang nhiệt độ
      // LM35: 10mV/°C, ADC 10-bit với Vref=5V
      temperature = (adc_value * 5.0 * 100.0) / 1024.0;
      
      // In kết quả
      printf("ADC: %ld, Nhiệt độ: %.1f°C\r\n", 
             adc_value, temperature);
      
      delay_ms(1000);
   }
}
```

### 4.2. Đo điện áp nhiều kênh
```c
#include <16F877A.h>
#device ADC=10
#use delay(crystal=20000000)
#use rs232(baud=9600, xmit=PIN_C6, rcv=PIN_C7)

void main() {
   float voltage[4];
   int16 adc_value;
   int8 channel;
   
   // Cấu hình ADC
   setup_adc_ports(ALL_ANALOG);
   setup_adc(ADC_CLOCK_DIV_32);
   
   while(1) {
      // Quét 4 kênh đầu tiên
      for(channel=0; channel<4; channel++) {
         set_adc_channel(channel);
         delay_us(10);
         
         adc_value = read_adc();
         voltage[channel] = (adc_value * 5.0) / 1024.0;
         
         printf("Kênh AN%u: %.2fV\r\n", 
                channel, voltage[channel]);
      }
      
      printf("\r\n");
      delay_ms(1000);
   }
}
```

### 4.3. Bài tập thực hành

1. **Cảm biến ánh sáng**
```c
// Sử dụng quang trở với điện trở phân áp
// Kết nối với AN0
void main() {
   int16 light_value;
   float voltage;
   
   setup_adc_ports(AN0_AN1_VSS_VREF);
   setup_adc(ADC_CLOCK_INTERNAL);
   
   while(1) {
      set_adc_channel(0);
      delay_us(10);
      light_value = read_adc();
      
      voltage = (light_value * 5.0) / 1024.0;
      printf("Ánh sáng: %ld, Điện áp: %.2fV\r\n", 
             light_value, voltage);
             
      delay_ms(500);
   }
}
```

### 4.4. Các lưu ý quan trọng

1. **Thời gian ổn định**
   - Đợi ít nhất 10µs sau khi chuyển kênh
   - Tăng thời gian đợi nếu có tụ lọc

2. **Nhiễu và độ chính xác**
   - Sử dụng tụ bypass 0.1µF gần chân VDD
   - Tránh đặt dây tín hiệu analog gần nguồn nhiễu
   - Cân nhắc sử dụng phương pháp lấy trung bình

3. **Tối ưu hóa**
   ```c
   // Lấy trung bình nhiều lần đọc
   int16 read_adc_avg(int8 channel, int8 samples) {
      int32 sum = 0;
      int8 i;
      
      set_adc_channel(channel);
      delay_us(10);
      
      for(i=0; i<samples; i++) {
         sum += read_adc();
         delay_us(20);
      }
      
      return sum / samples;
   }
   ```

---
© 2024 Embedded Systems Education. Tài liệu này được tạo cho mục đích giáo dục.
