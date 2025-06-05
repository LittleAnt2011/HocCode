# Đề 9: Đếm số lần xuất hiện của dãy bit 0b1101 trong N
# N = SW[9:0] (10-bit)

.equ LEDS, 0xFF200000
.equ SWITCHS, 0xFF200040

.text
.globl _start

_start:
    li sp, 0x03FFFFFC                    # Khởi tạo stack

main_loop:
    li t0, SWITCHS
    lw t1, 0(t0)                         # Đọc switch
    andi a0, t1, 0x3FF                   # N = SW[9:0] (10-bit mask)
    
    jal ra, dem_day_bit_1101             # Gọi hàm đếm dãy bit
    
    li t0, LEDS
    sw a0, 0(t0)                         # Xuất kết quả ra LED
    
    j main_loop                          # Lặp lại liên tục

# Hàm đếm số lần xuất hiện của dãy bit 0b1101
# Input: a0 = N (10-bit)
# Output: a0 = số lần xuất hiện của 0b1101
dem_day_bit_1101:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)                        # s0 = N
    sw s1, 8(sp)                         # s1 = count
    sw s2, 4(sp)                         # s2 = sliding window (4-bit)
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = N
    li s1, 0                             # count = 0
    li s2, 0                             # sliding window = 0
    li t0, 10                            # số bit cần kiểm tra
    
    # Xử lý trường hợp đặc biệt: N = 0
    bne s0, zero, dem_start
    j dem_end
    
dem_start:
    # Lấy 3 bit đầu vào window trước
    li t1, 3                             # lấy 3 bit đầu
    
dem_init_loop:
    bne t1, zero, dem_init_continue
    j dem_main_loop
    
dem_init_continue:
    slli s2, s2, 1                       # Shift window trái 1 bit
    andi s2, s2, 0xF                     # Mask 4-bit
    andi t2, s0, 1                       # Lấy bit LSB của N
    or s2, s2, t2                        # Thêm bit vào window
    srli s0, s0, 1                       # Shift N phải 1 bit
    addi t1, t1, -1                      # Giảm counter
    addi t0, t0, -1                      # Giảm số bit còn lại
    j dem_init_loop
    
dem_main_loop:
    bne t0, zero, dem_continue           # Còn bit để xử lý
    j dem_end
    
dem_continue:
    # Shift window và thêm bit mới
    slli s2, s2, 1                       # Shift window trái 1 bit
    andi s2, s2, 0xF                     # Mask 4-bit
    andi t2, s0, 1                       # Lấy bit LSB của N
    or s2, s2, t2                        # Thêm bit vào window
    
    # Kiểm tra window có bằng 0b1101 = 13 không
    li t3, 13                            # 0b1101 = 13
    bne s2, t3, dem_next                 # Nếu không bằng thì next
    addi s1, s1, 1                       # Tăng count
    
dem_next:
    srli s0, s0, 1                       # Shift N phải 1 bit
    addi t0, t0, -1                      # Giảm số bit còn lại
    j dem_main_loop
    
dem_end:
    mv a0, s1                            # Trả về count
    
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret