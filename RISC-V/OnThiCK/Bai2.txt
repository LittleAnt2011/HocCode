# Đề 2: Tính N! với N là số 4-bit từ SW[3:0]

.equ LEDS, 0xFF200000
.equ SWITCHS, 0xFF200040

.text
.globl _start

_start:
    li sp, 0x03FFFFFC                    # Khởi tạo stack

main_loop:
    li t0, SWITCHS
    lw t1, 0(t0)                         # Đọc switch
    andi a0, t1, 0xF                     # Mask 4-bit N từ SW[3:0]
    
    jal ra, tinh_giai_thua               # Gọi hàm tính N!
    
    li t0, LEDS
    sw a0, 0(t0)                         # Xuất kết quả ra LED
    
    j main_loop                          # Lặp lại liên tục

# Hàm tính giai thừa N! (không dùng lệnh nhân)
tinh_giai_thua:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)
    sw s1, 8(sp)
    sw s2, 4(sp)
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = N
    li s1, 1                             # s1 = kết quả = 1
    
    ble s0, zero, tinh_giai_thua_end     # Nếu N <= 0 thì trả về 1
    
tinh_giai_thua_loop:
    mv a0, s1                            # a0 = result hiện tại
    mv a1, s0                            # a1 = N hiện tại
    jal ra, nhan_hai_so                  # result = result * N
    mv s1, a0                            # Lưu kết quả
    
    addi s0, s0, -1                      # N--
    bgt s0, zero, tinh_giai_thua_loop    # Lặp nếu N > 0
    
tinh_giai_thua_end:
    mv a0, s1                            # Trả về kết quả
    
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret

# Hàm nhân hai số (dùng First Multiply Algorithm)
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
    beqz t1, nhan_skip_add               # Nếu bit = 0 thì skip add
    
    add s2, s2, s0                       # 1a: product = product + multiplicand
    
nhan_skip_add:
    slli s0, s0, 1                       # 2: Shift left multiplicand
    srli s1, s1, 1                       # 3: Shift right multiplier
    
    addi t0, t0, -1                      # Giảm counter
    bnez t0, nhan_loop                   # Lặp 32 lần
    
    mv a0, s2                            # Trả về product
    
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret