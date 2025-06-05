# Đề 6: Phép nhân theo lưu đồ version 2
# multiplicand x12 = SW[3:0], multiplier x13 = SW[7:4], product x11
# x11 = a1, x12 = a2, x13 = a3 (theo RISC-V calling convention)

.equ LEDS, 0xFF200000
.equ SWITCHS, 0xFF200040

.text
.globl _start

_start:
    li sp, 0x03FFFFFC                    # Khởi tạo stack

main_loop:
    li t0, SWITCHS
    lw t1, 0(t0)                         # Đọc switch
    andi a2, t1, 0xF                     # x12 = SW[3:0] (multiplicand)
    srli t1, t1, 4
    andi a3, t1, 0xF                     # x13 = SW[7:4] (multiplier)
    
    # Input: a2 = multiplicand (x12), a3 = multiplier (x13)
    # Output: a1 = product (x11)
    jal ra, nhan_theo_luu_do_v2          # Gọi hàm nhân theo lưu đồ version 2
    
    li t0, LEDS
    sw a1, 0(t0)                         # Xuất product x11 ra LED
    
    j main_loop                          # Lặp lại liên tục

# Hàm nhân theo lưu đồ version 2 (theo hình trong đề 6)
# Thuật toán: Test Multiplier0, nếu = 1 thì add multiplicand vào left half của product
# Input: a2 = multiplicand (x12), a3 = multiplier (x13)
# Output: a1 = product (x11)
nhan_theo_luu_do_v2:
    addi sp, sp, -20                     # Prologue
    sw s0, 16(sp)                        # s0 = multiplicand
    sw s1, 12(sp)                        # s1 = multiplier
    sw s2, 8(sp)                         # s2 = product (64-bit, chỉ dùng 32-bit thấp)
    sw s3, 4(sp)                         # s3 = product high (left half)
    sw ra, 0(sp)
    
    mv s0, a2                            # s0 = multiplicand (x12)
    mv s1, a3                            # s1 = multiplier (x13)
    li s2, 0                             # product low = 0
    li s3, 0                             # product high = 0
    li t0, 32                            # counter = 32 repetitions
    
nhan_v2_loop:
    # 1. Test Multiplier0 (bit LSB của multiplier)
    andi t1, s1, 1                       # Kiểm tra bit LSB của multiplier
    bne t1, zero, nhan_v2_add            # Nếu bit = 1 thì add
    j nhan_v2_shift                      # Nếu bit = 0 thì skip add
    
nhan_v2_add:
    # 1a. Add multiplicand to the left half of product
    add s3, s3, s0                       # product_high += multiplicand
    
nhan_v2_shift:
    # 2. Shift the Product register right 1 bit (64-bit shift)
    andi t2, s3, 1                       # Lấy bit LSB của product_high
    srli s3, s3, 1                       # Shift right product_high
    slli t2, t2, 31                      # Đưa bit LSB lên MSB position
    srli s2, s2, 1                       # Shift right product_low
    or s2, s2, t2                        # Nối bit từ product_high xuống product_low
    
    # 3. Shift the Multiplier register right 1 bit
    srli s1, s1, 1                       # Shift right multiplier
    
    addi t0, t0, -1                      # Giảm counter
    bne t0, zero, nhan_v2_loop           # Lặp 32 lần
    
nhan_v2_end:
    mv a1, s2                            # Gán product vào x11 (a1)
    
    lw ra, 0(sp)                         # Epilogue
    lw s3, 4(sp)
    lw s2, 8(sp)
    lw s1, 12(sp)
    lw s0, 16(sp)
    addi sp, sp, 20
    
    ret

# TEST CASES:
# SW[7:4]=0010, SW[3:0]=0011 (x13=2, x12=3): LED = 0x006   2×3=6
# SW[7:4]=0100, SW[3:0]=0101 (x13=4, x12=5): LED = 0x014   4×5=20
# SW[7:4]=0011, SW[3:0]=0111 (x13=3, x12=7): LED = 0x015   3×7=21
# SW[7:4]=1000, SW[3:0]=0010 (x13=8, x12=2): LED = 0x010   8×2=16
# SW[7:4]=1111, SW[3:0]=0001 (x13=15, x12=1): LED = 0x00F  15×1=15
# SW[7:4]=0000, SW[3:0]=1111 (x13=0, x12=15): LED = 0x000  0×15=0

# THUẬT TOÁN THEO LƯU ĐỒ VERSION 2:
# 1. Khởi tạo: product = 0 (64-bit: high + low), counter = 32
# 2. Loop:
#    - Test bit 0 của multiplier
#    - Nếu bit = 1: product_high = product_high + multiplicand
#    - Shift right product (64-bit): high → low
#    - Shift right multiplier 1 bit
#    - counter--
# 3. Lặp cho đến hết 32 lần  
# 4. Return product_low trong x11 (a1)
#
# KHÁC BIỆT VỚI VERSION 1:
# - Version 1: Shift left multiplicand, product cố định
# - Version 2: Multiplicand cố định, shift right product (64-bit)