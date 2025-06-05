# Đề 3: Tính UCLN của N và M
# N từ SW[3:0], M từ SW[7:4]
# x11 = UCLN(N, M) và x11 được gán vào a1

.equ LEDS, 0xFF200000                    # Định nghĩa địa chỉ LED (memory-mapped I/O)
.equ SWITCHS, 0xFF200040                 # Định nghĩa địa chỉ Switch (memory-mapped I/O)

.text                                    # Bắt đầu section code
.globl _start                            # Khai báo label _start global

_start:
    li sp, 0x03FFFFFC                    # Khởi tạo stack pointer = top of stack

main_loop:
    li t0, SWITCHS                       # Load địa chỉ switch vào t0
    lw t1, 0(t0)                         # Đọc giá trị switch từ memory vào t1
    andi a0, t1, 0xF                     # Mask 4 bit thấp → N = SW[3:0]
    srli t1, t1, 4                       # Shift right 4 bit để lấy SW[7:4]
    andi a1, t1, 0xF                     # Mask 4 bit thấp → M = SW[7:4]
    
    jal ra, tinh_ucln                    # Gọi hàm tính UCLN, lưu return address
    
    # Kết quả UCLN nằm trong a1 (x11)
    li t0, LEDS                          # Load địa chỉ LED vào t0
    sw a1, 0(t0)                         # Ghi giá trị x11 (a1) ra LED
    
    j main_loop                          # Jump vô hạn về main_loop

# Hàm tính UCLN (thuật toán Euclid)
# Input: a0 = N, a1 = M
# Output: a1 = UCLN(N, M) - x11
tinh_ucln:
    addi sp, sp, -12                     # Tạo stack frame: sp = sp - 12
    sw s0, 8(sp)                         # Lưu s0 vào stack (offset 8)
    sw s1, 4(sp)                         # Lưu s1 vào stack (offset 4)
    sw ra, 0(sp)                         # Lưu return address vào stack (offset 0)
    
    mv s0, a0                            # Copy N từ a0 vào s0 (preserved register)
    mv s1, a1                            # Copy M từ a1 vào s1 (preserved register)
    
    # Xử lý trường hợp đặc biệt
    bne s0, zero, check_m                # Nếu N ≠ 0, jump tới check_m
    mv a1, s1                            # Nếu N = 0, thì x11 = M
    j ucln_end                           # Jump tới kết thúc hàm
    
check_m:
    bne s1, zero, ucln_loop              # Nếu M ≠ 0, jump tới ucln_loop
    mv a1, s0                            # Nếu M = 0, thì x11 = N
    j ucln_end                           # Jump tới kết thúc hàm
    
ucln_loop:                               # Vòng lặp chính thuật toán Euclid
    beq s0, s1, ucln_found               # Nếu N = M, tìm được UCLN
    blt s0, s1, ucln_swap                # Nếu N < M, jump tới ucln_swap
    
    sub s0, s0, s1                       # N = N - M (giảm số lớn hơn)
    j ucln_loop                          # Jump lại đầu vòng lặp
    
ucln_swap:                               # Xử lý khi N < M
    sub s1, s1, s0                       # M = M - N (giảm số lớn hơn)
    j ucln_loop                          # Jump lại đầu vòng lặp
    
ucln_found:                              # Tìm được UCLN
    mv a1, s0                            # Gán kết quả UCLN vào x11 (a1)
    
ucln_end:                                # Kết thúc hàm
    lw ra, 0(sp)                         # Khôi phục return address từ stack
    lw s1, 4(sp)                         # Khôi phục s1 từ stack
    lw s0, 8(sp)                         # Khôi phục s0 từ stack
    addi sp, sp, 12                      # Giải phóng stack frame: sp = sp + 12
    
    ret                                  # Return về caller (jr ra)

# TEST CASES:
# SW[7:4]=0011, SW[3:0]=0010 (M=3, N=2): LED = 0x001  UCLN(2,3)=1
# SW[7:4]=0100, SW[3:0]=0110 (M=4, N=6): LED = 0x002  UCLN(6,4)=2
# SW[7:4]=0110, SW[3:0]=1001 (M=6, N=9): LED = 0x003  UCLN(9,6)=3
# SW[7:4]=1000, SW[3:0]=1100 (M=8, N=12): LED = 0x004 UCLN(12,8)=4
# SW[7:4]=0101, SW[3:0]=0111 (M=5, N=7): LED = 0x001  UCLN(7,5)=1
# SW[7:4]=0000, SW[3:0]=0101 (M=0, N=5): LED = 0x005  UCLN(5,0)=5