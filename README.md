Clone from https://github.com/espressif/esp-drone

## ESP-Drone firmware Build
1. 編譯ESP-IDF 需要以下軟件包

```
sudo apt-get install git wget flex bison gperf python3 python3-pip python3-setuptools cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
```

2. 設置Python 3 為Rpi3 的默認Python 版本

```
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 10 && alias pip=pip3
```
3. 獲取ESP-IDF (建議先用 v4.2 , 因為最新的 V4.4 會有system auto restart issue)

```
mkdir -p ~/esp
cd ~/esp
git clone -b release/v4. --recursive https://github.com/espressif/esp-idf.git
```

4. 安裝ESP-IDF 使用的各種工具

```
cd ~/esp/esp-idf
./install.sh
```

5. 設置環境變量

```
. $HOME/esp/esp-idf/export.sh
```

6. 先測試 Hello_World ,要先 確定可以build
```
cd ~/esp
cp -r $IDF_PATH/examples/get-started/hello_world .
cd ~/esp/hello_world
idf.py set-target esp32s2
idf.py build
```
7. 請確認有產生 hello-world.bin 及以下的message
```
$ idf.py build
Running cmake in directory /path/to/hello_world/build
Executing "cmake -G Ninja --warn-uninitialized /path/to/hello_world"...
Warn about uninitialized values.
-- Found Git:/usr/bin/git (found version "2.17.0")
-- Building empty aws_iot component due to configuration
-- Component names: ...
-- Component paths: ...

... (more lines of build system output)

[527/527] Generating hello-world.bin
esptool.py v2.3.1

Project build complete. To flash, run this command:
../../../components/esptool_py/esptool/esptool.py -p (PORT) -b 921600 write_flash --flash_mode dio --flash_size detect --flash_freq 40m 0x10000 build/hello-world.bin  build 0x1000 build/bootloader/bootloader.bin 0x8000 build/partition_table/partition-table.bin
or run 'idf.py -p PORT flash'
```

8. 修改以下鏈接腳本 (注意：沒加會導致 ESP-Drone compiler failed)
 ${IDF_PATH}/components/esp32/ld/esp32.project.ld.in  ${IDF_PATH}/components/esp32s2/ld/esp32s2.project.ld.in
 將以下代碼放在的末尾.flash.rodata。
```
/* Parameters and log system data */
 _param_start = .;
 KEEP(*(.param))
 KEEP(*(.param.*))
 _param_stop = .;
 . = ALIGN(4);
 _log_start = .;
 KEEP(*(.log))
 KEEP(*(.log.*))
 _log_stop = .;
 . = ALIGN(4);
 ```
 
 ![](https://i.imgur.com/cJssoCP.png)


9. Get Esp-Drone code 
* 因為最新的 code有相容性問題, 無法使用Crazyflie cfcclient, 所以建議退到前一版.
```
git clone https://github.com/espressif/esp-drone
git branch newBranch 673175cc10a456d9249c64b3fe8d90686c1a7875 
git checkout newBranch
```
10. 請參考此git code 修改wifi channel to 11. (wifi_config.ap.channel  = 11;)

11. Build Esp-Drone 

```
cd esp-drone
idf.py set-target esp32s2
idf.py build
```
預期結果:
![](https://i.imgur.com/g6YFTbg.png)

先 backup 原本的Rom image, 刷掛還可以用此binary 還原 
```
esptool.py -p /dev/ttyUSB0 -b 921600 read_flash 0 0x400000 OrgEspDronedump.bin
```

Flash: (example: your ESP-Drone mount to /dev/ttyUSB0)
```
  ipy.py -p /dev/ttyUSB0 flash
```

如果燒錄失敗，請用以下命令救援ESP-Drone
```
esptool.py write_flash 0x0 OrgEspDronedump.bin
```

附上台灣Android /Rpi3/Rpi4 可以使用的版本 
```
esptool.py write_flash 0x0 EspDroneTW_Channel11.bin
```
