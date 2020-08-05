wpa_supplicant user manual
===========================


configure file
--------------
You need to edit configure file first before using wpa_supplicant. It needs to be executed as root identity. You can name it anything ending with .conf.
```text
moxa@Moxa:~$ sudo -i
[sudo] password for moxa: 
root@Moxa:~# vim /etc/wpa_supplicant/wpa_supplicant.conf 
```

Next, type in these two lines and save it.
```text
ctrl_interface=DIR=/run/wpa_supplicant
update_config=1
```


wpa_supplicant
--------------
Execute the following command.
```text
root@Moxa:~# wpa_supplicant -B -i wlp2s0 -c /etc/wpa_supplicant/wpa_supplicant.conf
Successfully initialized wpa_supplicant
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

Use scan and scan_result commands to list available APs.
```text
> scan
OK
<3>CTRL-EVENT-SCAN-STARTED
<3>CTRL-EVENT-SCAN-RESULTS
> scan_results
bssid / frequency / signal level / flags / ssid
7c:57:3c:2e:d5:f1       5180    -44     [WPA2-EAP-CCMP][ESS]    .M-Mobile
7c:57:3c:2e:d5:f2       5180    -44     [WPA2-PSK-CCMP][ESS]    .M-Guest
74:da:da:35:72:6e       5765    -50     [WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][WPS][ESS]       94-test-5GHz
74:da:da:35:72:6c       2437    -54     [WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][WPS][ESS]       94-test
```

Next step: add and set the new network. In this step, the variables you need to set depend on the AP you want to connect. By default, you don't need to set additional variables if your AP corresponds to the following table.

| variables | not set for default |
| --------- | ------------------- |
| proto | WPA or RSN |
| key management protocols | WPA-PSK or WPA-EAP |
| pairwise ciphers for WPA | CCMP or TKIP |
| group ciphers for WPA | CCMP or TKIP or WEP104 or WEP40 |

The link describes all variables used in wpa_supplicant.
https://www.daemon-systems.org/man/wpa_supplicant.conf.5.html

For instance, you want to connect a network which uses WPA/WPA2 as protocol, CCMP+TKIP as encryption method. Since WPA/WPA2 and CCMP+TKIP are default variables, you only need to set "SSID" and "PSK".

```text
> add_network
0
> set_network 0 ssid "94-test-5GHz"
OK
> set_network 0 psk "12345678"
OK
> enable_network 0   move to up section
OK
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
p2p_device_address=00:15:61:20:a2:f9
address=00:15:61:20:a2:f9
uuid=06e6bd41-31b2-5725-aea0-9de37e96668f
```

After you finish the configuration, you can save the settings simply by typing save_config.
```text
> save_config
OK
> quit
```

After you enter save_config command, wpa_cli will automatically save the preference to the configuration file. The following is the outcome of the configure file saved in /etc/wpa_supplicant/wpa_supplicant.conf. These messages were generated after you saved network variables in wpa_cli interface.

```text
ctrl_interface=DIR=/run/wpa_supplicant 
update_config=1 
 
network={ 
        ssid="94-test-5GHz" 
        psk="12345678" 
}
```
Interfaces setting
------------------
Bringing up wireless interface.
```text
root@MOXA:~# ip link set wlp2s0 up
```


Setting IP address
------------------
After setting up wpa_supplicant, the next step is to set up the IP address. To do this, you have two choices: manual or automatic.

To set IP address automatically, you only need to use dhclient command.  
```text
root@MOXA:~# dhclient wlp2s0
```
If you want to set it manually, you need to use IP command or see Network Settings. For example, if your ip address is 192.168.0.109, subnet-mask is 255.255.255.0, and gateway is 192.168.0.1, you can use the following instruction.

```text
root@MOXA:~# ip addr add 192.168.0.109/24 dev wlp2s0
root@MOXA:~# ip route add default via 192.168.0.1 dev wlp2s0 
```

