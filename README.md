# fridaypi2

Based on [hallard/RPI-Lora-Gateway - GitHub](https://github.com/hallard/RPI-Lora-Gateway)



## Raspberry Pi (Raspbian) 初始化步驟



### (1) 燒入 Raspbian image

在 PC 上執行如下命令來做出 Raspbian 的 SD 卡。

```
$ sudo dd if="2017-09-07-raspbian-stretch-lite.img" of="/dev/sdX" bs=2048
```



### (2) 第一次開機

把燒入好 Raspbian 的 SD 卡插進 Raspberry Pi2 而上電。
因爲第一次啓動 Raspbian 時自動被做擴大 SD 卡的分區，
所以會花比較多時間 (好幾分鐘) 。

設定網路之前，我是利用 USB/UART 轉頭器來登入
Raspbian (user=```pi```, passwd=```raspbian```) 。



#### (2-1) 新增 Wi-Fi 用界面

因爲這次是用 USB Wi-Fi dongle 來上網,
所以首先要新增 ```wlan0``` 用的 Network 界面。

```console
$ sudo touch /etc/network/interfaces.d/wlan0
$ sudo vi /etc/network/interfaces.d/wlan0
```

```/etc/network/interfaces.d/wlan0``` 的內容是如下引用。

```
allow-hotplug wlan0
iface wlan0 inet dhcp
  wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```



#### (2-2) 設定 SSID 與 WPA 的密碼

設定基本的資訊。

```console
$ sudo vi /etc/wpa_supplicant/wpa_supplicant.conf
```

```diff
--- /etc/wpa_supplicant/wpa_supplicant.conf.0	2017-09-07 16:05:58.099999735 +0000
+++ /etc/wpa_supplicant/wpa_supplicant.conf	2017-10-01 02:49:50.693425111 +0000
@@ -1,3 +1,11 @@
-country=GB
+country=TW
 ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
-update_config=1
+update_config=0
}}}
```

後面新增 SSID 與其密碼。

```console
$ wpa_passphrase SSID PASSPHRASE | sudo tee --append /etc/wpa_supplicant/wpa_supplicant.conf
```



#### (2-3) 新增 udev rule

設定好這個 udev rule 之後，隨時可以插拔 USB Wi-Fi dongle 。

```console
$ sudo touch /etc/udev/rules.d/70-wlan0.rules
$ sudo vi /etc/udev/rules.d/70-wlan0.rules
```

```/etc/udev/rules.d/70-wlan0.rules``` 的內容是如下引用。

```
KERNEL="wlan0", ENV{SYSTEMD_WANTS}+="ifup@wlan.service"
```


#### (2-4) 生效 sshd

```console
$ sudo touch /boot/ssh
```



#### (2-6) 音響系統停止

這次不需要音響系統。

```console
$ sudo touch /etc/modprobe.d/blacklist-snd_bcm2835.conf
$ sudo vi /etc/modprobe.d/blacklist-snd_bcm2835.conf
```

```/etc/modprobe.d/blacklist-snd_bcm2835.conf``` 的內容是如下引用。

```
blacklist snd_bcm2835
```



#### (2-7) 新增 Avahi service (ssh)

```console
$ sudo cp /usr/share/doc/avahi-daemon/examples/ssh.service /etc/avahi/services
$ sudo service avahi-daemon restart
```

參考:

 * [Avahi - ArchWiki](https://wiki.archlinux.org/index.php/avahi)


#### (2-8) 停止 IPv6

```diff
--- /etc/avahi/avahi-daemon.conf.0	2017-10-02 11:14:43.530392820 +0000
+++ /etc/avahi/avahi-daemon.conf	2017-10-02 11:14:55.950400237 +0000
@@ -23,7 +23,7 @@
 #domain-name=local
 #browse-domains=0pointer.de, zeroconf.org
 use-ipv4=yes
-use-ipv6=yes
+use-ipv6=no
 #allow-interfaces=eth0
 #deny-interfaces=eth1
 #check-response-ttl=no
```

```diff
--- /etc/sysctl.conf.0	2017-10-02 11:21:00.650616647 +0000
+++ /etc/sysctl.conf	2017-10-02 11:22:14.480660272 +0000
@@ -75,3 +75,6 @@
 # See https://www.kernel.org/doc/Documentation/sysctl/fs.txt
 #fs.protected_hardlinks=0
 #fs.protected_symlinks=0
+
+net.ipv6.conf.all.disable_ipv6 = 1
+net.ipv6.conf.default.disable_ipv6 = 1
```

```console
$ sudo sysctl --load
```



#### (2-9) 重開機

```console
$ sudo reboot
```



### (3) 第二次開機以後……

以後不需要接 USB/UART 轉頭器。
插著 USB Wi-Fi dongle 開機，開機之後把它插進去，都可以。
開機完成後在遠端利用 ```ssh``` 就可以操作 Raspbian 。



## 編譯軟體



### lib2835.a

 * [C library for Broadcom BCM 2835 as used in Raspberry Pi](http://www.airspayce.com/mikem/bcm2835/)

```console
$ cd
$ wget http://www.airspayce.com/mikem/bcm2835/bcm2835-1.52.tar.gz
$ gunzip -c bcm2835-1.52.tar.gz | tar xvf -
$ cd bcm2835-1.52
$ ./configure
$ make -j4
$ sudo make check
$ sudo make install
```



### RadioHead

 * [hallard/RadioHead - GitHub](https://github.com/hallard/RadioHead)



#### Example: spi_scan

```console
$ cd
$ git clone https://github.com/hallard/RadioHead.git
$ cd RadioHead/examples/raspi/spi_scan
$ make
g++ -DRASPBERRY_PI -DBCM2835_NO_DELAY_COMPATIBILITY -c -I../../.. spi_scan.c
g++ spi_scan.o -lbcm2835  -o spi_scan
$ sudo ./spi_scan 
$ sudo ./spi_scan
Checking register(0x42) with CS=GPIO06 => Nothing!
Checking register(0x10) with CS=GPIO06 => Nothing!
Checking register(0x42) with CS=GPIO07 => Nothing!
Checking register(0x10) with CS=GPIO07 => Nothing!
Checking register(0x42) with CS=GPIO08 => SX1276 RF95/96 (V=0x12)
Checking register(0x10) with CS=GPIO08 => Nothing!
Checking register(0x42) with CS=GPIO26 => Nothing!
Checking register(0x10) with CS=GPIO26 => Nothing!
```



#### Example: multi_server

做 patch 。

```diff
--- multi_server.cpp.0	2017-11-12 06:54:24.682667170 +0000
+++ multi_server.cpp	2017-11-12 07:12:27.495111896 +0000
@@ -33,7 +33,7 @@
 
 // Our RF95 module 1 Configuration 
 #define RF95_1_NODE_ID    1
-#define RF95_1_FREQUENCY  868.00
+#define RF95_1_FREQUENCY  920.00
 
 // Our RF95 module 3 Configuration 
 #define RF95_2_NODE_ID    1
@@ -49,12 +49,13 @@
 
 // Modules table index
 #define NB_MODULES 3
+#define ENABLED_NB_MODULES 1
 uint8_t IRQ_pins[NB_MODULES] = { MOD1_IRQ_PIN, MOD2_IRQ_PIN, MOD3_IRQ_PIN};
 uint8_t LED_pins[NB_MODULES] = { MOD1_LED_PIN, MOD2_LED_PIN, MOD3_LED_PIN};
 uint8_t RST_pins[NB_MODULES] = { MOD1_RST_PIN, MOD2_RST_PIN, MOD3_RST_PIN};
 uint8_t CSN_pins[NB_MODULES] = { MOD1_CS_PIN , MOD2_CS_PIN , MOD3_CS_PIN };
 
-const char * MOD_name[]      = { "1 RF95 868",     "2 RF95 433" ,    "3 RFM69HW 433"  };
+const char * MOD_name[]      = { "1 RF95 920",     "2 RF95 433" ,    "3 RFM69HW 433"  };
 float MOD_freq[NB_MODULES]   = { RF95_1_FREQUENCY, RF95_2_FREQUENCY, RF69_3_FREQUENCY };
 uint8_t MOD_id[NB_MODULES]   = { RF95_1_NODE_ID,   RF95_2_NODE_ID,   RF69_3_NODE_ID   };
 
@@ -276,14 +277,14 @@
 
   // configure all modules I/O CS pins to 1 before anything else
   // to avoid any problem with SPI sharing
-  for (uint8_t i=0 ; i<NB_MODULES; i++) {
+  for (uint8_t i=0 ; i<ENABLED_NB_MODULES; i++) {
     // CS Ping as output and set to 1
     pinMode(CSN_pins[i], OUTPUT);
     digitalWrite(CSN_pins[i], HIGH);
   }
 
   // configure all modules I/O pins 
-  for (uint8_t i=0 ; i<NB_MODULES; i++) {
+  for (uint8_t i=0 ; i<ENABLED_NB_MODULES; i++) {
     // configure all modules
     if (!initRadioModule(i)){
       force_exit = true;
@@ -293,17 +294,26 @@
   // All init went fine, continue specific init if any
   if (!force_exit) {
     // Set all modules in receive mode
-    rf95_1.setModeRx();
-    rf95_2.setModeRx();
-    rf69_3.setModeRx();
-    printf( "Listening for incoming packets...\n" );
+    uint8_t i=ENABLED_NB_MODULES;
+    if (i--) { 
+      rf95_1.setModeRx();
+      printf( "Listening[0] for incoming packets...\n");
+      if (i --) {
+        rf95_2.setModeRx();
+        printf( "Listening[0] for incoming packets...\n");
+        if (i --) {
+          rf69_3.setModeRx();
+          printf( "Listening[0] for incoming packets...\n");
+        }
+      }
+    }
   }
 
   // Begin the main loop code 
   // ========================
   while (!force_exit) { 
     // Loop thru modules
-    for (uint8_t idx=0 ; idx<NB_MODULES ; idx++) {
+    for (uint8_t idx=0 ; idx<ENABLED_NB_MODULES ; idx++) {
       // Rising edge fired ?
       if (bcm2835_gpio_eds(IRQ_pins[idx])) {
         // Now clear the eds flag by setting it to 1
@@ -339,7 +349,7 @@
   // Light off on board LED
   digitalWrite(LED_PIN, LOW);
   // All module LEDs off, all modules CS line High
-  for (uint8_t i=0 ; i<NB_MODULES; i++) {
+  for (uint8_t i=0 ; i<ENABLED_NB_MODULES; i++) {
     digitalWrite(LED_pins[i], LOW);
     digitalWrite(CSN_pins[i], HIGH);
   }
```

```console
$ cd
$ cd RadioHead/examples/raspi/multi_server
$ make 
g++ -DRASPBERRY_PI -DBCM2835_NO_DELAY_COMPATIBILITY -D__BASEFILE__=\"multi_server\" -c -I../../.. multi_server.cpp
g++ -DRASPBERRY_PI -DBCM2835_NO_DELAY_COMPATIBILITY -D__BASEFILE__=\"RasPi\" -c ../../../RHutil/RasPi.cpp -I../../..
g++ -DRASPBERRY_PI -DBCM2835_NO_DELAY_COMPATIBILITY -D__BASEFILE__=\"RHHardwareSPI\" -c -I../../.. ../../../RHHardwareSPI.cpp
g++ -DRASPBERRY_PI -DBCM2835_NO_DELAY_COMPATIBILITY -D__BASEFILE__=\"RH_RF69\" -c -I../../.. ../../../RH_RF69.cpp
g++ -DRASPBERRY_PI -DBCM2835_NO_DELAY_COMPATIBILITY -D__BASEFILE__=\"RH_RF95\" -c -I../../.. ../../../RH_RF95.cpp
g++ -DRASPBERRY_PI -DBCM2835_NO_DELAY_COMPATIBILITY -D__BASEFILE__=\"RHSPIDriver\" -c -I../../.. ../../../RHSPIDriver.cpp
g++ -DRASPBERRY_PI -DBCM2835_NO_DELAY_COMPATIBILITY -D__BASEFILE__=\"RHGenericDriver\" -c -I../../.. ../../../RHGenericDriver.cpp
g++ -DRASPBERRY_PI -DBCM2835_NO_DELAY_COMPATIBILITY -D__BASEFILE__=\"RHGenericSPI\" -c -I../../.. ../../../RHGenericSPI.cpp
g++ multi_server.o RasPi.o RHHardwareSPI.o RH_RF69.o RH_RF95.o RHSPIDriver.o RHGenericDriver.o RHGenericSPI.o -lbcm2835 -o multi_server
$ sudo ./multi_server 
multi_server
============
1 RF95 920 (CS=GPIO8, IRQ=GPIO25, RST=GPIO5, LED=GPIO4) OK!, NodeID=1 @ 920.00MHz
Listening[0] for incoming packets...
```



### Arduino-LMIC

 * [hallard/arduino-lmic - GitHub](https://github.com/hallard/arduino-lmic)

TBD.

