wpa_supplicant user mannual
===========================


configure file
--------------
You need to edit configure file first before using wpa_supplicant. It needs to be executed as root identity. You can name it anything ended with .conf.
```text
moxa@Moxa:~$ sudo -i
[sudo] password for moxa: 
root@Moxa:~# vim /etc/wpa_supplicant/wpa_supplicant.conf 
```

Second, type in these two lines and save it.
```text
ctrl_interface=DIR=/run/wpa_supplicant
update_config=1
```


wpa_supplicant
--------------
Execute the following command. The failure about the board can be ignored.
```text
root@Moxa:~# wpa_supplicant -B -i wlp2s0 -c /etc/wpa_supplicant/wpa_supplicant.conf
Successfully initialized wpa_supplicant
[  153.295257] ath10k_pci 0000:02:00.0: failed to get board id:-95
```

wpa_cli
-------
Enter the wpa client interface.
```text
root@Moxa:~# wpa_cli -i wlp2s0
wpa_cli v2.4
Copyright (c) 2004-2015, Jouni Malinen <j@w1.fi> and contributors

This software may be distributed under the terms of the BSD license.
See README for more details.



Interactive mode

>
```

Using scan and scan_result commands to list available APs.
```text
> scan
OK
<3>CTRL-EVENT-SCAN-STARTED
<3>CTRL-EVENT-SCAN-RESULTS
> scan_results
bssid / frequency / signal level / flags / ssid
7c:57:3c:2e:d5:f1       5180    -44     [WPA2-EAP-CCMP][ESS]    .M-Mobile
7c:57:3c:2e:d5:f2       5180    -44     [WPA2-PSK-CCMP][ESS]    .M-Guest
74:da:da:35:72:6e       5765    -50     [WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][                               WPS][ESS]       94-test-5GHz
74:da:da:35:72:6c       2437    -54     [WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][                               WPS][ESS]       94-test
```

Next step: adding and setting new network. In this step, the variables you need to set depend on the AP you want to connect. In default, you don't need to set additional variables if your AP correspond to the following table.

| variables | not set for default |
| --- | ------------------- |
| proto | WPA or RSN |
| key management protocols | WPA-PSK or WPA-EAP |
| pairwise ciphers for WPA | CCMP or TKIP |
| group ciphers for WPA  | CCMP or TKIP or WEP104 or WEP40 |
From the result of previous action, we know that 94-test-5GHz is WPA RSN, WPA-PSK, CCMP+TKIP. As a result, we don't need to add extra informations in this case.
The link describes all variables used in wpa_supplicant.
https://www.daemon-systems.org/man/wpa_supplicant.conf.5.html
```text
> add_network
0
> set_network 0 ssid "94-test-5GHz"
OK
> set_network 0 psk "12345678"
OK
```

Then, checking networks and enable it
```text
> list_networks
network id / ssid / bssid / flags
0       94-test-5GHz    any     [DISSABLE]
> enable_network 0
OK
<3>CTRL-EVENT-SCAN-STARTED
<3>CTRL-EVENT-SCAN-RESULTS
<3>WPS-AP-AVAILABLE
<3>SME: Trying to authenticate with 74:da:da:35:72:6e (SSID='94-test-5GHz' freq=5765 MHz)
<3>Trying to associate with 74:da:da:35:72:6e (SSID='94-test-5GHz' freq=5756 MHz)
<3>Associated with 74:da:da:35:72:6e
<3>WPA: Key negotiation completed with 74:da:da:35:72:6e [PTK=CCMP GTK=TKIP]
<3>CTRL-EVENT-CONNECTED - Connection to 74:da:da:35:72:6e completed [id=0 id_str=]
> list_networks
network id / ssid / bssid / flags
0       94-test-5GHz    any     [CURRENT]
> save_config
OK
> q
```


Interfaces setting
------------------
Bringing down wired interface and Bringing up wireless interface
```text
root@MOXA:~# ip link set enp1s0 down
root@MOXA:~# ip link set wlp2s0 up
```


Setting IP address
------------------
```text
root@MOXA:~# dhclient wlp2s0
```


Testing and Result
------------------
To verify the connection, you can use ping command and status command in wpa_cli.
```text
root@Moxa:~# ping google.com
PING google.com (216.58.200.238) 56(84) bytes of date.
64 bytes from tsa03s01-in-f14.le100.net (216.58.200.238): icmp_seq=1 ttl=115 time=10.2 ms 
64 bytes from tsa03s01-in-f14.le100.net (216.58.200.238): icmp_seq=1 ttl=115 time=14.3 ms 
64 bytes from tsa03s01-in-f14.le100.net (216.58.200.238): icmp_seq=1 ttl=115 time=7.38 ms 
^c
--- google.com ping statistics ---
3 packets transmitted, 9 received, 0% packet loss, time 3488 ms
rtt min/avg/max/mdev = 5.789/15.607/53.519/13.871 ms
root@Moxa:~# wpa_cli -i wlp2s0
> status
bssid=74:da:da:35:72:6e
freq=5765
ssid=94-test-5GHz
id=0
mode=station
pairwise_cipher=CCMP
group_cipher=TKIP
key_mgmt=WPA2-PSK
wpa_state=COMPLETED
ip_address=192.168.0.109
p2p_device_address=00:15:61:20:a2:f9
address=00:15:61:20:a2:f9
uuid=06e6bd41-31b2-5725-aea0-9de37e96668f
```
