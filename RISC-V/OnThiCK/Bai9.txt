.equ SWITCHES,	0xff200040
.equ LEDS,	0xff200000
.global _start

_start:
	li sp,0x03fffffc
Loop:
	li t0,SWITCHES
	lw t1,0(t0)
	# N 10-bit từ SW[9:0]
	andi a0,t1,0x3ff    # Mask 10 bit đầu
	# Gọi hàm đếm
	jal ra,DemDayBit1101
	# Xuất kết quả ra LED
	li t0,LEDS
	sw a0,0(t0)
	j Loop

DemDayBit1101:
	# Input: a0 = số N 10-bit
	# Output: a0 = số lần xuất hiện của 0b1101
	addi sp,sp,-16
	sw ra,0(sp)
	sw s0,4(sp)
	sw s1,8(sp)
	sw s2,12(sp)
	
	mv s0,a0        # s0 = N (số cần kiểm tra)
	li s1,0         # s1 = counter (đếm số lần xuất hiện)
	li s2,0xd       # s2 = pattern 0b1101 = 13 = 0xd
	
	# Kiểm tra từ bit 0 đến bit 6 (7 vị trí có thể có dãy 4 bit)
	# Với 10 bit: có thể kiểm tra 7 vị trí (0->6)
	li t0,7         # t0 = số lần lặp
	li t1,0         # t1 = vị trí hiện tại
	
KiemTra_Loop:
	beq t1,t0,KetThuc   # Nếu đã kiểm tra hết 7 vị trí
	
	# Lấy 4 bit từ vị trí t1
	srl t2,s0,t1        # Dịch phải t1 bit
	andi t2,t2,0xf      # Lấy 4 bit cuối
	
	# So sánh với pattern 0b1101
	beq t2,s2,TimThay   # Nếu match thì tăng counter
	j TiepTuc
	
TimThay:
	addi s1,s1,1        # Tăng counter
	
TiepTuc:
	addi t1,t1,1        # Tăng vị trí
	j KiemTra_Loop
	
KetThuc:
	mv a0,s1            # Trả về số lần đếm được
	
	lw ra,0(sp)
	lw s0,4(sp)
	lw s1,8(sp)
	lw s2,12(sp)
	addi sp,sp,16
	ret