# Đề 15: Tính A+B và hiển thị HEX[1:0] dạng thập phân
# A = SW[3:0], B = SW[7:4]

.data
SEVEN_SEG_DECODE_TABLE: 
    .byte 0b00111111, 0b00000110, 0b01011011, 0b01001111
    .byte 0b01100110, 0b01101101, 0b01111101, 0b00000111
    .byte 0b01111111, 0b01100111, 0b00000000, 0b00000000
    .byte 0b00000000, 0b00000000, 0b00000000, 0b00000000

.equ LEDS, 0xFF200000
.equ SWITCHS, 0xFF200040
.equ HEX_BASE, 0xFF200020

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
    
    jal ra, addHaiSo                     # Gọi hàm cộng A+B
    
    # Hiển thị kết quả trên HEX[1:0] dạng thập phân
    mv a1, a0                            # a1 = A+B
    jal ra, hien_thi_hex_thap_phan       # Hiển thị trên HEX
    
    j main_loop                          # Lặp lại liên tục

# Hàm cộng hai số theo định dạng: addHaiSo(int A, int B)
# Input: a0 = A, a1 = B
# Output: a0 = A + B
addHaiSo:
    addi sp, sp, -12                     # Prologue
    sw s0, 8(sp)                         # s0 = A
    sw s1, 4(sp)                         # s1 = B
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = A
    mv s1, a1                            # s1 = B
    
    add a0, s0, s1                       # a0 = A + B
    
    lw ra, 0(sp)                         # Epilogue
    lw s1, 4(sp)
    lw s0, 8(sp)
    addi sp, sp, 12
    
    ret

# Hàm hiển thị số thập phân trên HEX[1:0]
# Input: a1 = số cần hiển thị (0-30, vì max A+B = 15+15)
# Output: Hiển thị trên HEX[1:0]
hien_thi_hex_thap_phan:
    addi sp, sp, -16                     # Prologue
    sw s0, 12(sp)                        # s0 = số ban đầu
    sw s1, 8(sp)                         # s1 = hàng chục
    sw s2, 4(sp)                         # s2 = hàng đơn vị
    sw ra, 0(sp)
    
    mv s0, a1                            # s0 = số cần hiển thị
    
    # Tách hàng chục và đơn vị (số / 10, số % 10)
    li s1, 0                             # hàng chục = 0
    mv s2, s0                            # hàng đơn vị = số ban đầu
    
    # Tính hàng chục bằng cách trừ 10 liên tiếp
    li t0, 10
tach_hang_chuc:
    blt s2, t0, tach_xong                # s2 < 10 thì xong
    sub s2, s2, t0                       # s2 -= 10
    addi s1, s1, 1                       # hàng chục++
    j tach_hang_chuc
    
tach_xong:
    # Chuyển đổi sang mã 7 đoạn
    mv a0, s1                            # Chuyển hàng chục
    jal ra, chuyen_doi_7_doan
    mv t1, a0                            # t1 = mã 7 đoạn hàng chục
    
    mv a0, s2                            # Chuyển hàng đơn vị
    jal ra, chuyen_doi_7_doan
    mv t2, a0                            # t2 = mã 7 đoạn hàng đơn vị
    
    # Kết hợp HEX1 (hàng chục) và HEX0 (hàng đơn vị)
    slli t1, t1, 8                       # HEX1 ở bits [15:8]
    or t1, t1, t2                        # Kết hợp HEX1 và HEX0
    
    # Ghi ra HEX display
    li t0, HEX_BASE
    sw t1, 0(t0)
    
    lw ra, 0(sp)                         # Epilogue
    lw s2, 4(sp)
    lw s1, 8(sp)
    lw s0, 12(sp)
    addi sp, sp, 16
    
    ret

# Hàm chuyển đổi số (0-9) sang mã 7 đoạn sử dụng bảng lookup
# Input: a0 = số (0-9)
# Output: a0 = mã 7 đoạn
chuyen_doi_7_doan:
    addi sp, sp, -8                      # Prologue
    sw s0, 4(sp)
    sw ra, 0(sp)
    
    mv s0, a0                            # s0 = số cần chuyển đổi
    
    # Kiểm tra phạm vi hợp lệ (0-9)
    li t0, 9
    bgt s0, t0, chuyen_doi_invalid       # > 9 thì invalid
    blt s0, zero, chuyen_doi_invalid     # < 0 thì invalid
    
    # Lấy địa chỉ bảng và cộng offset
    la t0, SEVEN_SEG_DECODE_TABLE        # t0 = địa chỉ bảng
    add t0, t0, s0                       # t0 = địa chỉ bảng + index
    lb a0, 0(t0)                         # Load byte từ bảng
    j chuyen_doi_end
    
chuyen_doi_invalid:
    li a0, 0b00000000                    # Tắt hết nếu invalid
    
chuyen_doi_end:
    lw ra, 0(sp)                         # Epilogue
    lw s0, 4(sp)
    addi sp, sp, 8
    
    ret

# TEST CASES:
# SW[7:4]=0000, SW[3:0]=0000 (B=0, A=0): HEX = "00"   0+0=0
# SW[7:4]=0001, SW[3:0]=0010 (B=1, A=2): HEX = "03"   2+1=3
# SW[7:4]=0101, SW[3:0]=0111 (B=5, A=7): HEX = "12"   7+5=12
# SW[7:4]=1001, SW[3:0]=0110 (B=9, A=6): HEX = "15"   6+9=15
# SW[7:4]=1111, SW[3:0]=1111 (B=15, A=15): HEX = "30" 15+15=30
# SW[7:4]=1000, SW[3:0]=0010 (B=8, A=2): HEX = "10"   2+8=10
# SW[7:4]=0100, SW[3:0]=0011 (B=4, A=3): HEX = "07"   3+4=7