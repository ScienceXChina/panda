PATH := esp-open-sdk/xtensa-lx106-elf/bin:$(PATH)
CC = xtensa-lx106-elf-gcc
CFLAGS = -Iinclude/ -I. -mlongcalls -Iesp-open-sdk/ESP8266_NONOS_SDK_V1.5.4_16_05_20/driver_lib/include -std=c99 -DICACHE_FLASH
LDLIBS = -nostdlib -Wl,--start-group -lmain -lnet80211 -lwpa -llwip -lpp -lphy -Wl,--end-group -lgcc -ldriver -Wl,--gc-sections
LDFLAGS = -Teagle.app.v6.ld
OBJCP = xtensa-lx106-elf-objcopy
SDK_BASE = esp-open-sdk/ESP8266_NONOS_SDK_V1.5.4_16_05_20

flash: user1.bin
	#./tools/esptool.py write_flash 0 $(SDK_BASE)/bin/boot_v1.5.bin 0x01000 user1.bin 0x81000 user2.bin 0x3FE000 $(SDK_BASE)/bin/blank.bin
	./tools/esptool.py write_flash 0 $(SDK_BASE)/bin/boot_v1.5.bin 0x01000 user1.bin 0x3FE000 $(SDK_BASE)/bin/blank.bin

proxy-0x00000.bin: proxy
	./tools/esptool.py elf2image $^

proxy: proxy.o tcp_ota.o

proxy.o: proxy.c

tcp_ota.o: tcp_ota.c

oldflash: proxy-0x00000.bin
	./tools/esptool.py write_flash 0 proxy-0x00000.bin 0x40000 proxy-0x40000.bin

user1.bin: proxy.o tcp_ota.o st_ota.o elm327.o
	$(CC) $(CFLAGS) $^ -o a.out -L$(SDK_BASE)/ld -T$(SDK_BASE)/ld/eagle.app.v6.new.1024.app1.ld $(LDLIBS)
	$(OBJCP) --only-section .text -O binary a.out eagle.app.v6.text.bin
	$(OBJCP) --only-section .data -O binary a.out eagle.app.v6.data.bin
	$(OBJCP) --only-section .rodata -O binary a.out eagle.app.v6.rodata.bin
	$(OBJCP) --only-section .irom0.text -O binary a.out eagle.app.v6.irom0text.bin
	COMPILE=gcc python ./esp-open-sdk/ESP8266_NONOS_SDK_V1.5.4_16_05_20/tools/gen_appbin.py a.out 2 0 32 4 0
	rm -f eagle.app.v6.*.bin
	mv eagle.app.flash.bin $@

user2.bin: proxy.o tcp_ota.o st_ota.o elm327.o
	$(CC) $(CFLAGS) $^ -o a.out -L$(SDK_BASE)/ld -T$(SDK_BASE)/ld/eagle.app.v6.new.1024.app2.ld $(LDLIBS)
	$(OBJCP) --only-section .text -O binary a.out eagle.app.v6.text.bin
	$(OBJCP) --only-section .data -O binary a.out eagle.app.v6.data.bin
	$(OBJCP) --only-section .rodata -O binary a.out eagle.app.v6.rodata.bin
	$(OBJCP) --only-section .irom0.text -O binary a.out eagle.app.v6.irom0text.bin
	COMPILE=gcc python ./esp-open-sdk/ESP8266_NONOS_SDK_V1.5.4_16_05_20/tools/gen_appbin.py a.out 2 0 32 4 0
	rm -f eagle.app.v6.*.bin
	mv eagle.app.flash.bin $@

ota: user1.bin user2.bin
	./tools/tcp_flash.py 192.168.0.10 user1.bin user2.bin

clean:
	rm -f proxy proxy.o proxy-0x00000.bin proxy-0x40000.bin eagle.app.* user1.bin user2.bin a.out
