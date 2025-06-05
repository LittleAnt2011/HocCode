# Đề 7: Tính x11 = a² + 3b + 25
# a = SW[3:0], b = SW[7:4]

.equ LEDS, 0xFF200000
.equ SWITCHS, 0xFF200040

.text
.globl _start

_start:
    li sp, 0x03FFFFFC                    # Khởi tạo stack

main_loop:
    li t0, SWITCHS
    lw t1, 0(t0)                         # Đọc switch
    andi a0, t1, 0xF                     # a = SW[3:0]
    srli t1, t1, 4
    andi a1, t1, 0xF                     # b = SW[7:4]
    
    jal ra, tinh_bieu_thuc               # Gọi hàm tính biểu thức
    
    li t0, LEDS
    sw a0, 0(t0)                         # Xuất kết quả ra LED
    
    j main_loop                          # Lặp lại liên tục

# Hàm tính biểu thức x11 = a² + 3b + 25
# Input: a0 = a, a1 = b
# Output: a0 = a² + 3b + 25
tinh_bieu_thuc:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)                        # s0 = a
    sw s1, 8(sp)                         # s1 = b
    sw s2, 4(sp)                         # s2 = kết quả tổng
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = a
    mv s1, a1                            # s1 = b
    
    # Tính a² = a * a
    mv a0, s0                            # a0 = a
    mv a1, s0                            # a1 = a
    jal ra, nhan_hai_so                  # a0 = a²
    mv s2, a0                            # s2 = a²
    
    # Tính 3b = 3 * b
    li a0, 3                             # a0 = 3
    mv a1, s1                            # a1 = b
    jal ra, nhan_hai_so                  # a0 = 3b
    
    # Tính a² + 3b
    add s2, s2, a0                       # s2 = a² + 3b
    
    # Tính a² + 3b + 25
    addi s2, s2, 25                      # s2 = a² + 3b + 25
    
    mv a0, s2                            # Trả về kết quả
    
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret

# Hàm nhân hai số (dùng Shift-and-Add Algorithm)
# Input: a0 = multiplicand, a1 = multiplier
# Output: a0 = product
nhan_hai_so:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)                        # s0 = multiplicand
    sw s1, 8(sp)                         # s1 = multiplier
    sw s2, 4(sp)                         # s2 = product
    sw ra, 0(sp)
    
    mv s0, a0                            # multiplicand
    mv s1, a1                            # multiplier
    li s2, 0                             # product = 0
    li t0, 32                            # counter = 32 iterations
    
nhan_loop:
    andi t1, s1, 1                       # Kiểm tra bit LSB của multiplier
    bne t1, zero, nhan_add               # Nếu bit = 1 thì add
    j nhan_skip_add                      # Nếu bit = 0 thì skip
    
nhan_add:
    add s2, s2, s0                       # product = product + multiplicand
    
nhan_skip_add:
    slli s0, s0, 1                       # Shift left multiplicand
    srli s1, s1, 1                       # Shift right multiplier
    
    addi t0, t0, -1                      # Giảm counter
    bne t0, zero, nhan_loop              # Lặp 32 lần
    
    mv a0, s2                            # Trả về product
    
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret