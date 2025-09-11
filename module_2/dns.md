# Закрепляем DNS в системе (не дописан)
1) должен быть добавлен ip 127.0.0.53 в loopback
`sudo ip addr add 127.0.0.53/8 dev lo`
Спадает при ребуте!!!!
2) запущена служба
`sudo systemctl status systemd-resolved`
3) добавлены dns
`sudo nano /etc/systemd/resolved.conf`
DNS=8.8.8.8
FallbackDNS=1.1.1.1
#Domains=
#DNSSEC=no
#DNSOverTLS=no
#MulticastDNS=no
#LLMNR=no
#Cache=no-negative
#CacheFromLocalhost=no
#DNSStubListener=yes
#DNSStubListenerExtra=
#ReadEtcHosts=yes
#ResolveUnicastSingleLabel=no
