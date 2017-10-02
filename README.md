# fridaypi2

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
