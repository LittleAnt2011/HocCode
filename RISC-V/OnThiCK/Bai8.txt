# Đề 8: Đếm có bao nhiêu số chia cho 3 dư 1 trong A và B
# A = SW[3:0], B = SW[7:4]

.equ LEDS, 0xFF200000
.equ SWITCHS, 0xFF200040

.text
.globl _start

_start:
    li sp, 0x03FFFFFC                    # Khởi tạo stack

main_loop:
    li t0, SWITCHS
    lw t1, 0(t0)                         # Đọc switch
    andi a0, t1, 0xF                     # A = SW[3:0]
    srli t1, t1, 4
    andi a1, t1, 0xF                     # B = SW[7:4]
    
    jal ra, dem_chia_3_du_1              # Gọi hàm đếm
    
    li t0, LEDS
    sw a0, 0(t0)                         # Xuất kết quả ra LED
    
    j main_loop                          # Lặp lại liên tục

# Hàm đếm có bao nhiêu số chia cho 3 dư 1 trong A và B
# Input: a0 = A, a1 = B
# Output: a0 = số lượng số chia cho 3 dư 1
dem_chia_3_du_1:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)                        # s0 = A
    sw s1, 8(sp)                         # s1 = B
    sw s2, 4(sp)                         # s2 = count
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = A
    mv s1, a1                            # s1 = B
    li s2, 0                             # count = 0
    
    # Kiểm tra A có chia cho 3 dư 1 không
    mv a0, s0                            # a0 = A
    jal ra, kiem_tra_chia_3_du_1         # Kiểm tra A
    add s2, s2, a0                       # count += kết quả (0 hoặc 1)
    
    # Kiểm tra B có chia cho 3 dư 1 không
    mv a0, s1                            # a0 = B
    jal ra, kiem_tra_chia_3_du_1         # Kiểm tra B
    add s2, s2, a0                       # count += kết quả (0 hoặc 1)
    
    mv a0, s2                            # Trả về count
    
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret

# Hàm kiểm tra một số có chia cho 3 dư 1 không
# Input: a0 = số cần kiểm tra
# Output: a0 = 1 nếu chia cho 3 dư 1, 0 nếu không
kiem_tra_chia_3_du_1:
    addi sp, sp, -12                     # Prologue
    sw s0, 8(sp)                         # s0 = số ban đầu
    sw s1, 4(sp)                         # s1 = số dư
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = số cần kiểm tra
    
    # Tính số % 3 bằng cách trừ liên tiếp
    mv s1, s0                            # s1 = số hiện tại
    
kiem_tra_loop:
    blt s1, zero, kiem_tra_negative      # Nếu s1 < 0
    li t0, 3
    blt s1, t0, kiem_tra_remainder       # Nếu s1 < 3 thì là số dư
    sub s1, s1, t0                       # s1 = s1 - 3
    j kiem_tra_loop
    
kiem_tra_negative:
    li a0, 0                             # Số âm thì trả về 0
    j kiem_tra_end
    
kiem_tra_remainder:
    # s1 chứa số dư (0, 1, hoặc 2)
    li t0, 1
    bne s1, t0, kiem_tra_not_1           # Nếu dư != 1
    li a0, 1                             # Dư 1 thì trả về 1
    j kiem_tra_end
    
kiem_tra_not_1:
    li a0, 0                             # Dư != 1 thì trả về 0
    
kiem_tra_end:
    lw ra, 0(sp)                         # Epilogue
    lw s1, 4(sp)
    lw s0, 8(sp)
    addi sp, sp, 12
    
    ret