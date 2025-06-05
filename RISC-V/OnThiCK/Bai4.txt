# Đề 4: Tính N^M
# N từ SW[3:0] (4-bit), M từ SW[5:4] (2-bit)
# x11 = N^M và x11 được gán vào a1

.equ LEDS, 0xFF200000
.equ SWITCHS, 0xFF200040

.text
.globl _start

_start:
    li sp, 0x03FFFFFC                    # Khởi tạo stack

main_loop:
    li t0, SWITCHS
    lw t1, 0(t0)                         # Đọc switch
    andi a0, t1, 0xF                     # N = SW[3:0]
    srli t1, t1, 4
    andi a1, t1, 0x3                     # M = SW[5:4] (2-bit)
    
    jal ra, tinh_luy_thua                # Gọi hàm tính N^M
    
    # Kết quả N^M nằm trong a1 (x11)
    li t0, LEDS
    sw a1, 0(t0)                         # Xuất x11 ra LED
    
    j main_loop                          # Lặp lại liên tục

# Hàm tính lũy thừa N^M
# Input: a0 = N, a1 = M
# Output: a1 = N^M (x11)
tinh_luy_thua:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)
    sw s1, 8(sp)
    sw s2, 4(sp)
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = N (base)
    mv s1, a1                            # s1 = M (exponent)
    li s2, 1                             # s2 = kết quả = 1
    
    # Xử lý trường hợp đặc biệt
    bne s1, zero, luy_thua_loop          # Nếu M != 0 thì bắt đầu tính
    mv a1, s2                            # Nếu M = 0 thì x11 = N^0 = 1
    j luy_thua_end                       

luy_thua_loop:
    mv a0, s2                            # a0 = kết quả hiện tại
    mv a1, s0                            # a1 = N
    jal ra, nhan_hai_so                  # kết quả = kết quả * N
    mv s2, a0                            # Lưu kết quả từ a0
    
    addi s1, s1, -1                      # M--
    bne s1, zero, luy_thua_loop          # Lặp M lần

luy_thua_end:
    mv a1, s2                            # Gán kết quả vào x11 (a1)
    
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

# TEST CASES:
# SW[5:4]=00, SW[3:0]=0101 (M=0, N=5): LED = 0x001   5^0=1
# SW[5:4]=01, SW[3:0]=0011 (M=1, N=3): LED = 0x003   3^1=3
# SW[5:4]=10, SW[3:0]=0010 (M=2, N=2): LED = 0x004   2^2=4
# SW[5:4]=10, SW[3:0]=0011 (M=2, N=3): LED = 0x009   3^2=9
# SW[5:4]=11, SW[3:0]=0010 (M=3, N=2): LED = 0x008   2^3=8
# SW[5:4]=11, SW[3:0]=0011 (M=3, N=3): LED = 0x01B   3^3=27