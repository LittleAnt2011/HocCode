
# Đề 12: Điều khiển dịch bit LED bằng KEY
# KEY[1]: dịch trái, KEY[2]: dịch phải, KEY[3]: đọc switch

.equ LEDS, 0xFF200000
.equ SWITCHS, 0xFF200040
.equ KEYS, 0xFF200050

.text
.globl _start

_start:
    li sp, 0x03FFFFFC                    # Khởi tạo stack
    li s0, 0                             # s0 = giá trị hiện tại trên LED

main_loop:
    jal ra, kiem_tra_nut_nhan            # Kiểm tra nút nào được nhấn
    
    # a0 chứa mã nút: 1=KEY[1], 2=KEY[2], 3=KEY[3], 0=không có
    li t0, 1
    beq a0, t0, xu_ly_key1               # KEY[1] - dịch trái
    li t0, 2
    beq a0, t0, xu_ly_key2               # KEY[2] - dịch phải
    li t0, 3
    beq a0, t0, xu_ly_key3               # KEY[3] - đọc switch
    j main_loop                          # Không có nút nào, lặp lại

xu_ly_key1:
    slli s0, s0, 1                       # Dịch trái 1 bit
    andi s0, s0, 0x3FF                   # Mask 10-bit cho LED[9:0]
    j cap_nhat_led

xu_ly_key2:
    srli s0, s0, 1                       # Dịch phải 1 bit
    j cap_nhat_led

xu_ly_key3:
    li t0, SWITCHS
    lw s0, 0(t0)                         # Đọc giá trị từ switch
    andi s0, s0, 0x3FF                   # Mask 10-bit
    j cap_nhat_led

cap_nhat_led:
    li t0, LEDS
    sw s0, 0(t0)                         # Cập nhật LED
    
    # Đợi thả nút (debounce)
doi_tha_nut:
    jal ra, kiem_tra_nut_nhan
    bne a0, zero, doi_tha_nut            # Còn nút được nhấn thì đợi
    
    j main_loop                          # Quay lại vòng lặp chính

# Hàm con kiểm tra nút nào được nhấn trong KEY[1], KEY[2], KEY[3]
# Output: a0 = 1 (KEY[1]), 2 (KEY[2]), 3 (KEY[3]), 0 (không có)
kiem_tra_nut_nhan:
    addi sp, sp, -8                      # Prologue
    sw s0, 4(sp)                         # s0 = trạng thái KEY
    sw ra, 0(sp)
    
    li t0, KEYS
    lw s0, 0(t0)                         # Đọc trạng thái KEY
    
    # Kiểm tra KEY[3] (bit 3)
    andi t1, s0, 0x8                     # Mask bit 3
    bne t1, zero, key3_pressed           # KEY[3] được nhấn
    
    # Kiểm tra KEY[2] (bit 2)
    andi t1, s0, 0x4                     # Mask bit 2
    bne t1, zero, key2_pressed           # KEY[2] được nhấn
    
    # Kiểm tra KEY[1] (bit 1)
    andi t1, s0, 0x2                     # Mask bit 1
    bne t1, zero, key1_pressed           # KEY[1] được nhấn
    
    # Không có nút nào được nhấn
    li a0, 0
    j kiem_tra_nut_end

key1_pressed:
    li a0, 1                             # Trả về mã KEY[1]
    j kiem_tra_nut_end

key2_pressed:
    li a0, 2                             # Trả về mã KEY[2]
    j kiem_tra_nut_end

key3_pressed:
    li a0, 3                             # Trả về mã KEY[3]
    j kiem_tra_nut_end

kiem_tra_nut_end:
    lw ra, 0(sp)                         # Epilogue
    lw s0, 4(sp)
    addi sp, sp, 8
    
    ret

# HƯỚNG DẪN SỬ DỤNG:
# 1. Nhấn KEY[3] để đọc giá trị từ switch vào LED
# 2. Nhấn KEY[1] để dịch pattern trên LED sang trái
# 3. Nhấn KEY[2] để dịch pattern trên LED sang phải
# 4. Pattern sẽ được giữ nguyên cho đến khi có thao tác mới

# TEST CASES:
# - Khởi động: LED = 0 (tất cả tắt)
# - Nhấn KEY[3] với SW = 0x155: LED = 0x155
# - Nhấn KEY[1]: LED = 0x2AA (dịch trái)
# - Nhấn KEY[1] tiếp: LED = 0x154 (dịch trái, overflow bit 10)
# - Nhấn KEY[2]: LED = 0x0AA (dịch phải)
# - Nhấn KEY[3] với SW = 0x001: LED = 0x001 (đọc switch mới)