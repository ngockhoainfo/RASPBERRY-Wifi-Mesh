# RASPBERRY-Wifi-Mesh

NOTE INSTALL RASPI STEP BY STEP
#1. Install the necessary software------------------------------
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install hostapd udhcpd dnsmasq -y
sudo apt-get install iptables -y

#2. Configure DHCP----------------------------------------------
sudo vi /etc/udhcpd.conf
start 192.168.42.2 
end 192.168.42.20
interface wlan0
remaining yes
opt dns 8.8.8.8 4.2.2.2
opt subnet 255.255.255.0
opt router 192.168.42.1
opt lease 864000

#3. Configure IP for Wlan0--------------------------------------
sudo ifconfig wlan0 192.168.42.1

#4. SETUP Station Interface for Rt5370-------------------
sudo vi /etc/network/interfaces
source-directory /etc/network/interfaces.d
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

iface wlan0 inet static
    address 192.168.42.1
    netmask 255.255.255.0
    wireless-power off

iface default inet dhcp
up iptables-restore < /etc/iptables.ipv4.nat

#5. SETUP AP Interface for Rt5370--------------------------
sudo vi /etc/network/interfaces.ap
source-directory /etc/network/interfaces.d
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

iface wlan0 inet static
    address 192.168.42.1
    netmask 255.255.255.0
    wireless-power off

iface default inet dhcp
up iptables-restore < /etc/iptables.ipv4.nat

#3. Configure HostAPD------------------------------------------------
sudo vi /etc/hostapd/hostapd.conf
interface=wlan0
driver=nl80211
ssid=group_12
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=0
wpa_passphrase=My_Passphrase
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

#4. Configure NAT--------------------------------------------
sudo vi /etc/iptables.ipv4.nat

*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i wlan0 -o eth0 -j ACCEPT
COMMIT
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o eth0 -j MASQUERADE
COMMIT

5. Start up-----------------------------------------------
DHCP & HOSTAPD & DNSMASQ
sudo service hostapd start
sudo service udhcpd start
sudo service dnsmasq start
hotspot to start
sudo update-rc.d hostapd enable
sudo update-rc.d udhcpd enable
sudo update-rc.d dnsmasq enable




