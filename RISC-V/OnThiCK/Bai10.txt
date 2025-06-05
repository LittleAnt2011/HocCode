# Đề 10: Kiểm tra N chia hết cho 0x11 và 0x18 < N < 0x40
# N = SW[9:2] (8-bit)

.equ LEDS, 0xFF200000
.equ SWITCHS, 0xFF200040

.text
.globl _start

_start:
    li sp, 0x03FFFFFC                    # Khởi tạo stack

main_loop:
    li t0, SWITCHS
    lw t1, 0(t0)                         # Đọc switch
    srli t1, t1, 2                       # Shift để lấy SW[9:2]
    andi a0, t1, 0xFF                    # N = SW[9:2] (8-bit mask)
    
    jal ra, kiem_tra_dieu_kien           # Gọi hàm kiểm tra
    
    li t0, LEDS
    sw a0, 0(t0)                         # Xuất kết quả ra LED
    
    j main_loop                          # Lặp lại liên tục

# Hàm kiểm tra điều kiện N
# Điều kiện 1: N chia hết cho 0x11 (17)
# Điều kiện 2: 0x18 < N < 0x40 (24 < N < 64)
# Input: a0 = N
# Output: a0 = LED pattern
kiem_tra_dieu_kien:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)                        # s0 = N
    sw s1, 8(sp)                         # s1 = điều kiện 1 (chia hết 0x11)
    sw s2, 4(sp)                         # s2 = điều kiện 2 (trong khoảng)
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = N
    li s1, 0                             # điều kiện 1 = false
    li s2, 0                             # điều kiện 2 = false
    
    # Kiểm tra điều kiện 1: N chia hết cho 0x11
    mv a0, s0
    jal ra, kiem_tra_chia_het_0x11       # Gọi hàm con
    mv s1, a0                            # s1 = kết quả điều kiện 1
    
    # Kiểm tra điều kiện 2: 0x18 < N < 0x40
    li t0, 0x18                          # 24
    li t1, 0x40                          # 64
    bgt s0, t0, check_upper_bound        # N > 24?
    j check_result                       # N <= 24, điều kiện 2 = false
    
check_upper_bound:
    blt s0, t1, condition_2_true         # N < 64?
    j check_result                       # N >= 64, điều kiện 2 = false
    
condition_2_true:
    li s2, 1                             # điều kiện 2 = true
    
check_result:
    # Xét kết quả:
    # Thỏa 1 trong 2: RED_LED[3:0] sáng (0xF)
    # Thỏa hết 2 điều kiện: RED_LED[7:4] sáng (0xF0)
    # Không thỏa: RED_LED[9:0] tắt (0x0)
    
    add t0, s1, s2                       # t0 = số điều kiện thỏa mãn
    
    bne t0, zero, check_both             # Có ít nhất 1 điều kiện thỏa
    li a0, 0                             # Không thỏa điều kiện nào
    j kiem_tra_end
    
check_both:
    li t1, 2
    bne t0, t1, one_condition            # Chỉ thỏa 1 điều kiện
    li a0, 0xF0                          # Thỏa cả 2 điều kiện: LED[7:4]
    j kiem_tra_end
    
one_condition:
    li a0, 0xF                           # Thỏa 1 điều kiện: LED[3:0]
    
kiem_tra_end:
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret

# Hàm con kiểm tra N có chia hết cho 0x11 không
# Input: a0 = N
# Output: a0 = 1 nếu chia hết, 0 nếu không
kiem_tra_chia_het_0x11:
    addi sp, sp, -12                     # Prologue
    sw s0, 8(sp)                         # s0 = N
    sw s1, 4(sp)                         # s1 = N copy để trừ
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = N
    mv s1, s0                            # s1 = N copy
    
    # Trừ liên tiếp 0x11 cho đến khi < 0x11
    li t0, 0x11                          # 17
    
chia_het_loop:
    blt s1, t0, chia_het_check           # s1 < 17?
    sub s1, s1, t0                       # s1 = s1 - 17
    j chia_het_loop
    
chia_het_check:
    bne s1, zero, chia_het_false         # s1 != 0 thì không chia hết
    li a0, 1                             # Chia hết
    j chia_het_end
    
chia_het_false:
    li a0, 0                             # Không chia hết
    
chia_het_end:
    lw ra, 0(sp)                         # Epilogue
    lw s1, 4(sp)
    lw s0, 8(sp)
    addi sp, sp, 12
    
    ret

# TEST CASES:
# SW[9:2] = 00100010 (N=34=0x22): LED = 0xF0   34%17=0 ✓, 24<34<64 ✓ → cả 2 điều kiện
# SW[9:2] = 00010001 (N=17=0x11): LED = 0xF    17%17=0 ✓, 24<17<64 ✗ → 1 điều kiện
# SW[9:2] = 00011110 (N=30=0x1E): LED = 0xF    30%17=13 ✗, 24<30<64 ✓ → 1 điều kiện
# SW[9:2] = 00001111 (N=15=0x0F): LED = 0x0    15%17=15 ✗, 24<15<64 ✗ → 0 điều kiện
# SW[9:2] = 01000000 (N=64=0x40): LED = 0xF    64%17=13 ✗, 24<64<64 ✗ → 0 điều kiện
# SW[9:2] = 00110011 (N=51=0x33): LED = 0xF0   51%17=0 ✓, 24<51<64 ✓ → cả 2 điều kiện