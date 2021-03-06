CROSS_COMPILE ?= arm-none-eabi-
CC := $(CROSS_COMPILE)gcc
AS := $(CROSS_COMPILE)as
CFLAGS = -fno-common -ffreestanding -O3 \
	 -gdwarf-2 -g3 -Wall -Werror \
	 -mcpu=cortex-m3 -mthumb \
	 -Wl,-Tos.ld -nostartfiles \

TARGET = os.bin
all: $(TARGET)

$(TARGET): main.c os.c startup.c context_switch.S
	$(CC) $(CFLAGS) $^ -o os.elf -std=c99
	$(CROSS_COMPILE)objcopy -Obinary os.elf os.bin
	$(CROSS_COMPILE)objdump -S os.elf > os.list

qemu: $(TARGET)
	@qemu-system-arm -M ? | grep stm32-p103 >/dev/null || exit
	@echo "Press Ctrl-A and then X to exit QEMU"
	@echo
	qemu-system-arm -M stm32-p103 -nographic -kernel os.bin

qemudbg: $(TARGET)
	qemu-system-arm -M stm32-p103 \
	    -monitor stdio\
	    -gdb tcp::3333 -S \
	    -kernel os.bin -semihosting 2>&1>/dev/null & \
	echo $$! > qemu_pid && \
	$(CROSS_COMPILE)gdb -x gdbscript && \
	cat qemu_pid | `xargs kill 2>/dev/null || test true` && \
	rm -f qemu_pid

clean:
	rm -f *.o *.elf *.bin *.list
