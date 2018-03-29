# Raspberry PI Wi-Fi Hotspot Configuration

## Updating & Installing

Update & Upgrade the PI

```
sudo apt-get update
sudo apt-get upgrade
```

Install DHCPD and HOSTAPD

```
sudo apt-get install dnsmasq hostapd
```

Stop the newly added services while thay are configured

```
sudo systemctl stop dnsmasq   
sudo systemctl stop hostapd
````

## Configuring DHCPD Client

Configure the static IP addresses for eth0 and wlan0

```
sudo nano /etc/dhcpcd.conf
```

Add to the end of the file:

```
# Configure eth0 for LAN
interface eth0
  static ip_address=10.94.65.230/24
  static domain_name_servers=192.168.0.1 8.8.8.8
  static routers=10.94.65.254

# Configure wlan0 for WiFi
interface wlan0
  static ip_address=192.168.4.1/24
```

---

Restart the DHCPCD service to apply the changes

```
sudo service dhcpcd restart
```

## Configuring DHCPD Server

Configure the range of IPs the server will issue out

```
sudo nano /etc/dnsmasq.conf
```

Add to the end of the file:

```
interface=wlan0
  dhcp-range=192.168.69.2,192.168.69.22,255.255.255.0,24h
```

## Configuring HOSTAPD

Configuring the WiFi hotspot with HOSTAPD

```
sudo nano /etc/hostapd/hostapd.conf
```

Add to the end of the file:

```
interface=wlan0
driver=nl80211
ssid=Patkis-Air
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=patkis1234
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

---

Next we need to tell HOSTAPD where the configuration file is

```
sudo nano /etc/default/hostapd
```

Find the DAEMON_CONF file and change it to this:

```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

## Starting Up

Now we can start the services back up

```
sudo systemctl start hostapd
sudo systemctl start dnsmasq
```

## IP Routing Configuration

We will make out WiFi requests route to the LAN network

```
sudo nano /etc/sysctl.conf
```

Uncomment this line:

```
net.ipv4.ip_forward=1
```

---

Add a masquerade for outbound traffic on eth0:

```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

---

Save the iptables rule:

```
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

---

Make these rules apply on boot:

```
sudo nano /etc/rc.local
```

Add this right before the `exit 0` line:

```
iptables-restore < /etc/iptables.ipv4.nat
```

## Reboot

```
sudo reboot
```
