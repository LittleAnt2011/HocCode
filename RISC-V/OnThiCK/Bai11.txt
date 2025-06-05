# Đề 11: Kiểm tra N chia hết cho 0x14 và lớn hơn 0x41
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
    
    jal ra, kiem_tra_dieu_kien_v2        # Gọi hàm kiểm tra
    
    li t0, LEDS
    sw a0, 0(t0)                         # Xuất kết quả ra LED
    
    j main_loop                          # Lặp lại liên tục

# Hàm kiểm tra điều kiện N
# Điều kiện 1: N chia hết cho 0x14 (20)
# Điều kiện 2: N > 0x41 (N > 65)
# Input: a0 = N
# Output: a0 = LED pattern
kiem_tra_dieu_kien_v2:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)                        # s0 = N
    sw s1, 8(sp)                         # s1 = điều kiện 1 (chia hết 0x14)
    sw s2, 4(sp)                         # s2 = điều kiện 2 (> 0x41)
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = N
    li s1, 0                             # điều kiện 1 = false
    li s2, 0                             # điều kiện 2 = false
    
    # Kiểm tra điều kiện 1: N chia hết cho 0x14
    mv a0, s0
    jal ra, kiem_tra_chia_het_0x14       # Gọi hàm con
    mv s1, a0                            # s1 = kết quả điều kiện 1
    
    # Kiểm tra điều kiện 2: N > 0x41
    li t0, 0x41                          # 65
    bgt s0, t0, condition_2_true_v2      # N > 65?
    j check_result_v2                    # N <= 65, điều kiện 2 = false
    
condition_2_true_v2:
    li s2, 1                             # điều kiện 2 = true
    
check_result_v2:
    # Xét kết quả:
    # Thỏa cả 2 điều kiện: RED_LED[9:0] sáng (0x3FF)
    # Thỏa 1 trong 2: RED_LED[3:0] sáng (0xF)
    # Không thỏa: RED_LED[9:0] tắt (0x0)
    
    add t0, s1, s2                       # t0 = số điều kiện thỏa mãn
    
    bne t0, zero, check_both_v2          # Có ít nhất 1 điều kiện thỏa
    li a0, 0                             # Không thỏa điều kiện nào
    j kiem_tra_end_v2
    
check_both_v2:
    li t1, 2
    bne t0, t1, one_condition_v2         # Chỉ thỏa 1 điều kiện
    li a0, 0x3FF                         # Thỏa cả 2 điều kiện: LED[9:0]
    j kiem_tra_end_v2
    
one_condition_v2:
    li a0, 0xF                           # Thỏa 1 điều kiện: LED[3:0]
    
kiem_tra_end_v2:
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret

# Hàm con kiểm tra N có chia hết cho 0x14 không
# Input: a0 = N
# Output: a0 = 1 nếu chia hết, 0 nếu không
kiem_tra_chia_het_0x14:
    addi sp, sp, -12                     # Prologue
    sw s0, 8(sp)                         # s0 = N
    sw s1, 4(sp)                         # s1 = N copy để trừ
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = N
    mv s1, s0                            # s1 = N copy
    
    # Trừ liên tiếp 0x14 cho đến khi < 0x14
    li t0, 0x14                          # 20
    
chia_het_loop_v2:
    blt s1, t0, chia_het_check_v2        # s1 < 20?
    sub s1, s1, t0                       # s1 = s1 - 20
    j chia_het_loop_v2
    
chia_het_check_v2:
    bne s1, zero, chia_het_false_v2      # s1 != 0 thì không chia hết
    li a0, 1                             # Chia hết
    j chia_het_end_v2
    
chia_het_false_v2:
    li a0, 0                             # Không chia hết
    
chia_het_end_v2:
    lw ra, 0(sp)                         # Epilogue
    lw s1, 4(sp)
    lw s0, 8(sp)
    addi sp, sp, 12
    
    ret

# TEST CASES:
# SW[9:2] = 01010000 (N=80=0x50): LED = 0x3FF  80%20=0 ✓, 80>65 ✓ → cả 2 điều kiện
# SW[9:2] = 00010100 (N=20=0x14): LED = 0xF    20%20=0 ✓, 20>65 ✗ → 1 điều kiện
# SW[9:2] = 01000010 (N=66=0x42): LED = 0xF    66%20=6 ✗, 66>65 ✓ → 1 điều kiện
# SW[9:2] = 00001111 (N=15=0x0F): LED = 0x0    15%20=15 ✗, 15>65 ✗ → 0 điều kiện
# SW[9:2] = 00000000 (N=0): LED = 0xF          0%20=0 ✓, 0>65 ✗ → 1 điều kiện
# SW[9:2] = 01100100 (N=100=0x64): LED = 0x3FF 100%20=0 ✓, 100>65 ✓ → cả 2 điều kiện