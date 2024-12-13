# Bài giảng: Đo nhiệt độ với LM35 và hiển thị LCD sử dụng PIC16F877A

## I. Tổng quan hệ thống

### 1.1. Các thành phần chính
1. **Cảm biến LM35** (U2)
   - Cảm biến nhiệt độ độ chính xác cao
   - Đầu ra: 10mV/°C
   - Cấp nguồn: VDD (5V)

2. **Vi điều khiển PIC16F877A** (U1)
   - Xử lý tín hiệu analog từ LM35
   - Điều khiển hiển thị LCD
   - ADC 10-bit tích hợp

3. **LCD LM016L** (LCD1)
   - Màn hình LCD 16x2 ký tự
   - Giao tiếp 4-bit
   - Hiển thị nhiệt độ

## II. Sơ đồ kết nối chi tiết

### 2.1. Kết nối LM35
- Chân 1 (VDD): Nối với nguồn 5V
- Chân 2 (VOUT): Nối với RA0/AN0 (Pin 2 của PIC)
- Chân 3 (GND): Nối với mass

### 2.2. Kết nối LCD
1. **Các chân điều khiển**:
   - RS: Nối với RD4/PSP4 (Pin 27)
   - RW: Nối với RD5/PSP5 (Pin 28)
   - E: Nối với RD6/PSP6 (Pin 29)

2. **Các chân dữ liệu (4-bit mode)**:
   - D4: Nối với RD0/PSP0 (Pin 19)
   - D5: Nối với RD1/PSP1 (Pin 20)
   - D6: Nối với RD2/PSP2 (Pin 21)
   - D7: Nối với RD3/PSP3 (Pin 22)

### 2.3. Các kết nối khác
- Thạch anh: Nối với pins OSC1/CLKIN và OSC2/CLKOUT
- MCLR: Nối với VDD qua điện trở pull-up

## III. Code mẫu

### 3.1. Cấu hình ban đầu
```c
#include <16F877A.h>
#device ADC=10
#fuses HS,NOWDT,NOPROTECT,NOLVP
#use delay(crystal=20000000)

// Định nghĩa chân LCD
#define LCD_RS PIN_D4
#define LCD_RW PIN_D5
#define LCD_E  PIN_D6
#define LCD_D4 PIN_D0
#define LCD_D5 PIN_D1
#define LCD_D6 PIN_D2
#define LCD_D7 PIN_D3

// Include thư viện LCD
#include <lcd.c>
```

### 3.2. Chương trình chính
```c
void main() {
   float temperature;
   int16 adc_value;
   char temp_text[16];
   
   // Khởi tạo ADC
   setup_adc_ports(AN0_AN1_AN2_AN3_AN4);
   setup_adc(ADC_CLOCK_INTERNAL);
   
   // Khởi tạo LCD
   lcd_init();
   
   while(1) {
      // Đọc giá trị ADC từ LM35
      set_adc_channel(0);
      delay_us(10);
      adc_value = read_adc();
      
      // Tính nhiệt độ
      temperature = (adc_value * 5.0 * 100.0) / 1024.0;
      
      // Hiển thị lên LCD
      lcd_gotoxy(1, 1);
      lcd_putc("Nhiet do Phong:");
      lcd_gotoxy(1, 2);
      sprintf(temp_text, "%2.1f do C", temperature);
      lcd_putc(temp_text);
      
      delay_ms(1000);
   }
}
```

## IV. Các lưu ý quan trọng

### 4.1. Phần cứng
1. **Cấp nguồn**:
   - Nguồn 5V ổn định cho cả hệ thống
   - Tụ lọc nguồn đủ lớn
   - Tụ bypass gần các IC

2. **Kết nối LCD**:
   - Điều chỉnh độ tương phản bằng biến trở
   - Điện trở hạn dòng cho đèn nền
   - Mode 4-bit tiết kiệm chân I/O

3. **Cảm biến LM35**:
   - Đặt gần điểm cần đo
   - Tránh nguồn nhiệt nhiễu
   - Dây tín hiệu càng ngắn càng tốt

### 4.2. Phần mềm
1. **Xử lý ADC**:
   - Chờ thời gian ổn định sau khi chuyển kênh
   - Có thể lấy trung bình nhiều lần đọc
   - Kiểm tra giá trị hợp lệ

2. **Hiển thị LCD**:
   - Cập nhật LCD định kỳ
   - Xử lý định dạng hiển thị
   - Tránh nhấp nháy không cần thiết

## V. Cải tiến có thể thực hiện

1. **Thêm tính năng**:
```c
// Thêm cảnh báo nhiệt độ cao
if(temperature > 35.0) {
   output_high(PIN_C0);  // Bật LED cảnh báo
   lcd_gotoxy(14, 2);
   lcd_putc("!");        // Hiển thị dấu cảnh báo
}
```

2. **Lọc nhiễu**:
```c
float read_temp_average() {
   float sum = 0;
   for(int i=0; i<8; i++) {
      set_adc_channel(0);
      delay_us(10);
      sum += read_adc();
      delay_ms(1);
   }
   return (sum * 5.0 * 100.0) / (1024.0 * 8);
}
```

## VI. Bài tập thực hành

1. Thêm chức năng lưu nhiệt độ max/min
2. Thêm nút nhấn để chuyển đổi °C/°F
3. Tạo cảnh báo khi nhiệt độ vượt ngưỡng
4. Hiển thị đồ thị nhiệt độ đơn giản trên LCD

---
© 2024 Embedded Systems Education. Tài liệu này được tạo cho mục đích giáo dục.