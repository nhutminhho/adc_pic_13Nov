# Bài giảng: Tính toán độ phân giải ADC với ví dụ cảm biến LM35

## I. Giới thiệu về cảm biến LM35

### 1.1. Đặc điểm của LM35
- Là cảm biến nhiệt độ có độ chính xác cao
- Điện áp đầu ra tuyến tính với nhiệt độ
- Độ nhạy: 10mV/1°C
- Dải đo: -55°C đến +150°C
- Không cần hiệu chỉnh bên ngoài

### 1.2. Nguyên lý hoạt động
- Ví dụ: ở 25°C, đầu ra sẽ là: 25 × 10mV = 250mV
- Công thức tổng quát: Vout = Temperature × 10mV

## II. Tính toán độ phân giải ADC

### 2.1. Điện áp tham chiếu
```
VREF- = VSS = 0V (Điện áp tham chiếu âm)
VREF+ = VDD = 5V (Điện áp tham chiếu dương)
```

### 2.2. Tính Step Size (SS)
- Step Size là giá trị điện áp nhỏ nhất mà ADC có thể phân biệt được
- Công thức:
```
SS = (VREF+ - VREF-) / (2^n - 1)
   = 5000mV / (2^10 - 1)
   = 5000mV / 1023
   = 4.887mV
```
Trong đó:
- 2^10 - 1 = 1023 (ADC 10 bit)
- VREF+ - VREF- = 5000mV (dải điện áp đầu vào)

### 2.3. Tính điện áp toàn thang (Full Scale)
```
FS = SS × (2^10 - 1)
   = 4.887mV × 1023
   = 5000mV
   = 5V
```

## III. Phân tích độ phân giải

### 3.1. So sánh độ phân giải
1. **Độ phân giải của ADC**: 4.887mV
2. **Độ phân giải của LM35**: 10mV/°C

### 3.2. Tính tỷ lệ chênh lệch
```
Tỷ lệ = 10mV / 4.887mV = 2.046
```

### 3.3. Ý nghĩa
- ADC có độ phân giải tốt hơn cảm biến (nhỏ hơn)
- Mỗi độ C sẽ tương ứng với khoảng 2 đơn vị ADC
- Không bị mất thông tin do độ phân giải ADC

## IV. Ứng dụng trong lập trình

### 4.1. Code mẫu đọc nhiệt độ
```c
#include <16F877A.h>
#device ADC=10
#use delay(crystal=20000000)

void main() {
   int16 adc_value;
   float temperature;
   
   setup_adc_ports(AN0_AN1_VSS_VREF);
   setup_adc(ADC_CLOCK_INTERNAL);
   
   while(1) {
      set_adc_channel(0);      // LM35 kết nối với AN0
      delay_us(10);            // Đợi ổn định
      
      adc_value = read_adc();
      
      // Công thức tính nhiệt độ có bù trừ tỷ lệ chênh lệch
      temperature = (adc_value * 4.887 / 10.0);
      // Hoặc đơn giản hơn:
      temperature = (adc_value * 5.0 * 100.0) / 1024.0;
      
      printf("Nhiệt độ: %.1f°C\n", temperature);
      delay_ms(1000);
   }
}
```

### 4.2. Tối ưu hóa độ chính xác
```c
float read_temperature() {
   int32 sum = 0;
   int8 i;
   
   // Đọc 8 lần và lấy trung bình
   for(i=0; i<8; i++) {
      set_adc_channel(0);
      delay_us(10);
      sum += read_adc();
      delay_ms(1);
   }
   
   return (sum * 5.0 * 100.0) / (1024.0 * 8);
}
```

## V. Các lưu ý quan trọng

### 5.1. Về phần cứng
1. **Nguồn cấp**:
   - Cần nguồn ổn định cho LM35
   - Sử dụng tụ lọc nguồn phù hợp

2. **Kết nối**:
   - Dây nối càng ngắn càng tốt
   - Tránh nguồn nhiễu điện từ

### 5.2. Về phần mềm
1. **Lọc nhiễu**:
   - Sử dụng kỹ thuật lấy trung bình nhiều lần đọc
   - Loại bỏ các giá trị bất thường

2. **Hiệu chỉnh**:
   - Có thể cần hiệu chỉnh offset
   - Định kỳ kiểm tra và hiệu chuẩn

## VI. Bài tập thực hành

1. **Bài tập 1**: Tính toán độ phân giải với VREF = 3.3V
2. **Bài tập 2**: Viết chương trình đo nhiệt độ và cảnh báo khi vượt ngưỡng
3. **Bài tập 3**: Cải thiện độ chính xác bằng kỹ thuật lọc nhiễu

---
© 2024 Embedded Systems Education. Tài liệu này được tạo cho mục đích giáo dục.
