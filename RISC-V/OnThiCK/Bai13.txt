# Đề 13: Kiểm tra số nguyên tố
# N = SW[7:0] (8-bit)

.equ LEDS, 0xFF200000
.equ SWITCHS, 0xFF200040

.text
.globl _start

_start:
    li sp, 0x03FFFFFC                    # Khởi tạo stack

main_loop:
    li t0, SWITCHS
    lw t1, 0(t0)                         # Đọc switch
    andi a0, t1, 0xFF                    # N = SW[7:0] (8-bit)
    
    jal ra, kiemTraSoNguyenTo            # Gọi hàm kiểm tra số nguyên tố
    
    li t0, LEDS
    bne a0, zero, tat_ca_led_sang        # Nếu là số nguyên tố
    sw zero, 0(t0)                       # Không phải số nguyên tố → tắt LED
    j main_loop
    
tat_ca_led_sang:
    li t1, 0x3FF                         # Tất cả LED[9:0] sáng
    sw t1, 0(t0)
    j main_loop

# Hàm kiểm tra số nguyên tố theo định dạng: bool kiemTraSoNguyenTo(int N)
# Input: a0 = N
# Output: a0 = 1 nếu là số nguyên tố, 0 nếu không
kiemTraSoNguyenTo:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)                        # s0 = N
    sw s1, 8(sp)                         # s1 = i (ước số)
    sw s2, 4(sp)                         # s2 = sqrt(N) xấp xỉ
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = N
    
    # Xử lý các trường hợp đặc biệt
    li t0, 2
    blt s0, t0, khong_phai_nguyen_to     # N < 2 → không phải nguyên tố
    beq s0, t0, la_nguyen_to             # N = 2 → là nguyên tố
    
    # Kiểm tra N có chẵn không (N > 2)
    andi t0, s0, 1                       # Kiểm tra bit LSB
    bne t0, zero, kiem_tra_le            # N lẻ → kiểm tra tiếp
    j khong_phai_nguyen_to               # N chẵn và > 2 → không phải nguyên tố
    
kiem_tra_le:
    # Tính sqrt(N) xấp xỉ bằng cách tìm i sao cho i*i <= N
    li s1, 3                             # Bắt đầu từ 3 (số lẻ đầu tiên > 2)
    mv s2, s0                            # s2 = upper bound
    
tim_sqrt_loop:
    mv a0, s1                            # a0 = i
    mv a1, s1                            # a1 = i
    jal ra, nhan_hai_so                  # a0 = i*i
    bgt a0, s0, kiem_tra_uoc             # i*i > N → bắt đầu kiểm tra ước
    addi s1, s1, 2                       # i += 2 (chỉ kiểm tra số lẻ)
    j tim_sqrt_loop
    
kiem_tra_uoc:
    li s1, 3                             # Reset i = 3
    
kiem_tra_uoc_loop:
    mv a0, s1                            # a0 = i
    mv a1, s1                            # a1 = i
    jal ra, nhan_hai_so                  # a0 = i*i
    bgt a0, s0, la_nguyen_to             # i*i > N → là nguyên tố
    
    # Kiểm tra N % i == 0
    mv a0, s0                            # a0 = N
    mv a1, s1                            # a1 = i
    jal ra, chia_lay_du                  # a0 = N % i
    bne a0, zero, kiem_tra_uoc_tiep      # N % i != 0 → kiểm tra ước tiếp
    j khong_phai_nguyen_to               # N % i == 0 → không phải nguyên tố
    
kiem_tra_uoc_tiep:
    addi s1, s1, 2                       # i += 2 (chỉ kiểm tra số lẻ)
    j kiem_tra_uoc_loop
    
la_nguyen_to:
    li a0, 1                             # Trả về true
    j kiem_tra_nguyen_to_end
    
khong_phai_nguyen_to:
    li a0, 0                             # Trả về false
    
kiem_tra_nguyen_to_end:
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret

# Hàm tính phép chia lấy dư (N % divisor)
# Input: a0 = N, a1 = divisor
# Output: a0 = N % divisor
chia_lay_du:
    addi sp, sp, -12                     # Prologue
    sw s0, 8(sp)                         # s0 = N
    sw s1, 4(sp)                         # s1 = divisor
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = N
    mv s1, a1                            # s1 = divisor
    
chia_lay_du_loop:
    blt s0, s1, chia_lay_du_end          # N < divisor → N là phần dư
    sub s0, s0, s1                       # N = N - divisor
    j chia_lay_du_loop
    
chia_lay_du_end:
    mv a0, s0                            # Trả về phần dư
    
    lw ra, 0(sp)                         # Epilogue
    lw s1, 4(sp)
    lw s0, 8(sp)
    addi sp, sp, 12
    
    ret

# Hàm nhân hai số
# Input: a0 = a, a1 = b
# Output: a0 = a * b
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
    add s2, s2, s0                       # product += multiplicand
    
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
# SW[7:0] = 00000010 (N=2): LED = 0x3FF   2 là số nguyên tố
# SW[7:0] = 00000011 (N=3): LED = 0x3FF   3 là số nguyên tố
# SW[7:0] = 00000100 (N=4): LED = 0x0     4 không phải số nguyên tố (4=2*2)
# SW[7:0] = 00000101 (N=5): LED = 0x3FF   5 là số nguyên tố
# SW[7:0] = 00001001 (N=9): LED = 0x0     9 không phải số nguyên tố (9=3*3)
# SW[7:0] = 00001011 (N=11): LED = 0x3FF  11 là số nguyên tố
# SW[7:0] = 00001111 (N=15): LED = 0x0    15 không phải số nguyên tố (15=3*5)
# SW[7:0] = 00010001 (N=17): LED = 0x3FF  17 là số nguyên tố