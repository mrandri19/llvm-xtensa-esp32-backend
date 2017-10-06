# ESP32 LLVM Backend notes
This is what `make flash` does:
```
python2 /home/andrea/esp/esp-idf/components/esptool_py/esptool/esptool.py
 --chip esp32
 --port /dev/ttyUSB0
 --baud 115200
 --before default_reset
 --after hard_reset write_flash -z 
 --flash_mode dio
 --flash_freq 40m
 --flash_size detect
 0x1000 /home/andrea/esp/hello_world/build/bootloader/bootloader.bin 
 0x10000 /home/andrea/esp/hello_world/build/hello-world.bin
 0x8000 /home/andrea/esp/hello_world/build/partitions_singleapp.bin
```

## Things to understand:
- How to create `hello-world.bin`
    - `make all` is called
    - `hello_world_main.c` is first mentioned in:
    - ```
      xtensa-esp32-elf-gcc -std=gnu99 -Og -ggdb -ffunction-sections -fdata-sections -fstrict-volatile-bitfields -mlongcalls -nostdlib -Wall -Werror=all -Wno-error=unused-function -Wno-error=unused-but-set-variable -Wno-error=unused-variable -Wno-error=deprecated-declarations -Wextra -Wno-unused-parameter -Wno-sign-compare -Wno-old-style-declaration -DWITH_POSIX -DMBEDTLS_CONFIG_FILE='"mbedtls/esp_config.h"' -DHAVE_CONFIG_H -DESP_PLATFORM -D IDF_VER=\"v3.0-dev-782-ge6afe28b\" -MMD -MP   -I /home/andrea/esp/hello_world/main/include -I /home/andrea/esp/esp-idf/components/app_trace/include -I /home/andrea/esp/esp-idf/components/app_update/include -I /home/andrea/esp/esp-idf/components/bootloader_support/include -I /home/andrea/esp/esp-idf/components/bt/include -I /home/andrea/esp/esp-idf/components/coap/port/include -I /home/andrea/esp/esp-idf/components/coap/port/include/coap -I /home/andrea/esp/esp-idf/components/coap/libcoap/include -I /home/andrea/esp/esp-idf/components/coap/libcoap/include/coap -I /home/andrea/esp/esp-idf/components/console/. -I /home/andrea/esp/esp-idf/components/cxx/include -I /home/andrea/esp/esp-idf/components/driver/include -I /home/andrea/esp/esp-idf/components/esp32/include -I /home/andrea/esp/esp-idf/components/esp_adc_cal/include -I /home/andrea/esp/esp-idf/components/ethernet/include -I /home/andrea/esp/esp-idf/components/expat/port/include -I /home/andrea/esp/esp-idf/components/expat/include/expat -I /home/andrea/esp/esp-idf/components/fatfs/src -I /home/andrea/esp/esp-idf/components/freertos/include -I /home/andrea/esp/esp-idf/components/heap/include -I /home/andrea/esp/esp-idf/components/jsmn/include/ -I /home/andrea/esp/esp-idf/components/json/include -I /home/andrea/esp/esp-idf/components/json/port/include -I /home/andrea/esp/esp-idf/components/libsodium/port_include -I /home/andrea/esp/esp-idf/components/libsodium/libsodium/src/libsodium/include -I /home/andrea/esp/esp-idf/components/log/include -I /home/andrea/esp/esp-idf/components/lwip/include/lwip -I /home/andrea/esp/esp-idf/components/lwip/include/lwip/port -I /home/andrea/esp/esp-idf/components/lwip/include/lwip/posix -I /home/andrea/esp/esp-idf/components/lwip/apps/ping -I /home/andrea/esp/esp-idf/components/mbedtls/port/include -I /home/andrea/esp/esp-idf/components/mbedtls/include -I /home/andrea/esp/esp-idf/components/mdns/include -I /home/andrea/esp/esp-idf/components/micro-ecc/micro-ecc -I /home/andrea/esp/esp-idf/components/newlib/platform_include -I /home/andrea/esp/esp-idf/components/newlib/include -I /home/andrea/esp/esp-idf/components/nghttp/port/include -I /home/andrea/esp/esp-idf/components/nghttp/nghttp2/lib/includes -I /home/andrea/esp/esp-idf/components/nvs_flash/include -I /home/andrea/esp/esp-idf/components/openssl/include -I /home/andrea/esp/esp-idf/components/pthread/include -I /home/andrea/esp/esp-idf/components/sdmmc/include -I /home/andrea/esp/esp-idf/components/soc/esp32/include -I /home/andrea/esp/esp-idf/components/soc/include -I /home/andrea/esp/esp-idf/components/spi_flash/include -I /home/andrea/esp/esp-idf/components/spiffs/include -I /home/andrea/esp/esp-idf/components/tcpip_adapter/include -I /home/andrea/esp/esp-idf/components/ulp/include -I /home/andrea/esp/esp-idf/components/vfs/include -I /home/andrea/esp/esp-idf/components/wear_levelling/include -I /home/andrea/esp/esp-idf/components/wpa_supplicant/include -I /home/andrea/esp/esp-idf/components/wpa_supplicant/port/include -I /home/andrea/esp/esp-idf/components/wpa_supplicant/../esp32/include -I /home/andrea/esp/esp-idf/components/xtensa-debug-module/include -I /home/andrea/esp/hello_world/build/include  -I. 
      -c  # -c means that gcc will not link the program and then output an object file
      /home/andrea/esp/hello_world/main/./hello_world_main.c
      -o hello_world_main.o
      ```
    - where it's compiled into an object file
    - `rm -f libmain.a`
    - `xtensa-esp32-elf-ar cru libmain.a hello_world_main.o`
    - Apparently `hello_world_main.o` is statically linked inside libmain.a
    - `libmain.a` is now in `/home/andrea/esp/hello_world/build/main`
    - then `hello-world.elf` is created
    - the last command used to create `hello-world.elf` is:
    - ```
      xtensa-esp32-elf-gcc -nostdlib -u call_user_start_cpu0  -Wl,--gc-sections -Wl,-static -Wl,--start-group  -L/home/andrea/esp/hello_world/build/app_trace -lapp_trace -L/home/andrea/esp/hello_world/build/app_update -lapp_update -L/home/andrea/esp/hello_world/build/aws_iot  -L/home/andrea/esp/hello_world/build/bootloader_support -lbootloader_support -L/home/andrea/esp/hello_world/build/bt -lbt -L/home/andrea/esp/hello_world/build/coap -lcoap -L/home/andrea/esp/hello_world/build/console -lconsole -L/home/andrea/esp/hello_world/build/cxx -lcxx -u __cxa_guard_dummy -L/home/andrea/esp/hello_world/build/driver -ldriver -L/home/andrea/esp/hello_world/build/esp32 -lesp32 /home/andrea/esp/esp-idf/components/esp32/libhal.a -L/home/andrea/esp/esp-idf/components/esp32/lib -lcore -lrtc -lnet80211 -lpp -lwpa -lsmartconfig -lcoexist -lwps -lwpa2 -lespnow -lphy -L /home/andrea/esp/esp-idf/components/esp32/ld -T esp32_out.ld -u ld_include_panic_highint_hdl -T esp32.common.ld -T esp32.rom.ld -T esp32.peripherals.ld -T esp32.rom.spiram_incompatible_fns.ld -L/home/andrea/esp/hello_world/build/esp_adc_cal -lesp_adc_cal -L/home/andrea/esp/hello_world/build/ethernet -lethernet -L/home/andrea/esp/hello_world/build/expat -lexpat -L/home/andrea/esp/hello_world/build/fatfs -lfatfs -L/home/andrea/esp/hello_world/build/freertos -lfreertos -Wl,--undefined=uxTopUsedPriority -L/home/andrea/esp/hello_world/build/heap -lheap -L/home/andrea/esp/hello_world/build/jsmn -ljsmn -L/home/andrea/esp/hello_world/build/json -ljson -L/home/andrea/esp/hello_world/build/libsodium -llibsodium -L/home/andrea/esp/hello_world/build/log -llog -L/home/andrea/esp/hello_world/build/lwip -llwip 

      -L/home/andrea/esp/hello_world/build/main
      -lmain
      # Here libmain.a is linked inside hello-world.elf

      -L/home/andrea/esp/hello_world/build/mbedtls -lmbedtls -L/home/andrea/esp/hello_world/build/mdns -lmdns -L/home/andrea/esp/hello_world/build/micro-ecc -lmicro-ecc -L/home/andrea/esp/hello_world/build/newlib /home/andrea/esp/esp-idf/components/newlib/lib/libc.a /home/andrea/esp/esp-idf/components/newlib/lib/libm.a -lnewlib -L/home/andrea/esp/hello_world/build/nghttp -lnghttp -L/home/andrea/esp/hello_world/build/nvs_flash -lnvs_flash -L/home/andrea/esp/hello_world/build/openssl -lopenssl -L/home/andrea/esp/hello_world/build/pthread -lpthread -L/home/andrea/esp/hello_world/build/sdmmc -lsdmmc -L/home/andrea/esp/hello_world/build/soc -lsoc -L/home/andrea/esp/hello_world/build/spi_flash -lspi_flash -L/home/andrea/esp/hello_world/build/spiffs -lspiffs -L/home/andrea/esp/hello_world/build/tcpip_adapter -ltcpip_adapter -L/home/andrea/esp/hello_world/build/ulp -lulp -L/home/andrea/esp/hello_world/build/vfs -lvfs -L/home/andrea/esp/hello_world/build/wear_levelling -lwear_levelling -L/home/andrea/esp/hello_world/build/wpa_supplicant -lwpa_supplicant -L/home/andrea/esp/hello_world/build/xtensa-debug-module -lxtensa-debug-module -lgcc -lstdc++ -lgcov -Wl,--end-group -Wl,-EL
      
      -o /home/andrea/esp/hello_world/build/hello-world.elf
      
      -Wl,-Map=/home/andrea/esp/hello_world/build/hello-world.map
        ```
    - which then is converted into an image by this command:
    - `python2 /home/andrea/esp/esp-idf/components/esptool_py/esptool/esptool.py --chip esp32 elf2image --flash_mode "dio" --flash_freq "40m" --flash_size "2MB"  -o /home/andrea/esp/hello_world/build/hello-world.bin /home/andrea/esp/hello_world/build/hello-world.elf`


- How to give our own `hello_world_main.o` file to the compilation process
