# Đề 5: Phép nhân theo lưu đồ
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
    jal ra, nhan_theo_luu_do             # Gọi hàm nhân theo lưu đồ
    
    li t0, LEDS
    sw a1, 0(t0)                         # Xuất product x11 ra LED
    
    j main_loop                          # Lặp lại liên tục

# Hàm nhân theo lưu đồ (Second Multiply Algorithm từ hình)
# Input: a2 = multiplicand (x12), a3 = multiplier (x13)
# Output: a1 = product (x11)
nhan_theo_luu_do:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)                        # s0 = multiplicand
    sw s1, 8(sp)                         # s1 = multiplier
    sw s2, 4(sp)                         # s2 = product
    sw ra, 0(sp)
    
    mv s0, a2                            # s0 = multiplicand (x12)
    mv s1, a3                            # s1 = multiplier (x13)
    li s2, 0                             # s2 = product = 0
    li t0, 32                            # counter = 32 repetitions
    
nhan_luu_do_loop:
    andi t1, s1, 1                       # Test bit LSB của multiplier
    bne t1, zero, nhan_luu_do_add        # Nếu bit = 1 thì add
    j nhan_luu_do_shift                  # Nếu bit = 0 thì skip add
    
nhan_luu_do_add:
    add s2, s2, s0                       # 1a: product = product + multiplicand
    
nhan_luu_do_shift:
    slli s0, s0, 1                       # 2: Shift left multiplicand 1 bit
    srli s1, s1, 1                       # 3: Shift right multiplier 1 bit
    
    addi t0, t0, -1                      # Giảm counter
    bne t0, zero, nhan_luu_do_loop       # Lặp 32 lần
    
nhan_luu_do_end:
    mv a1, s2                            # Gán product vào x11 (a1)
    
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret

# TEST CASES:
# SW[7:4]=0010, SW[3:0]=0011 (x13=2, x12=3): LED = 0x006   2×3=6
# SW[7:4]=0100, SW[3:0]=0101 (x13=4, x12=5): LED = 0x014   4×5=20
# SW[7:4]=0011, SW[3:0]=0111 (x13=3, x12=7): LED = 0x015   3×7=21
# SW[7:4]=1000, SW[3:0]=0010 (x13=8, x12=2): LED = 0x010   8×2=16
# SW[7:4]=1111, SW[3:0]=0001 (x13=15, x12=1): LED = 0x00F  15×1=15
# SW[7:4]=0000, SW[3:0]=1111 (x13=0, x12=15): LED = 0x000  0×15=0

# THUẬT TOÁN THEO LƯU ĐỒ:
# 1. Khởi tạo: product = 0, counter = 32
# 2. Loop:
#    - Test bit 0 của multiplier
#    - Nếu bit = 1: product = product + multiplicand  
#    - Shift left multiplicand 1 bit
#    - Shift right multiplier 1 bit
#    - counter--
# 3. Lặp cho đến hết 32 lần
# 4. Return product trong x11 (a1)