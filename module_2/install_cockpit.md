

- если не работает sudo apt update то пробрось dns
```bash
echo 'nameserver 8.8.8.8'| sudo tee /etc/resolv.conf
```
TODO закрепить dns для рестарта!


```bash
sudo apt -y install net-tools vim htop tree cockpit cockpit-pcp
```
```bash
sudo systemctl enable --now cockpit.socket
```

- in cockpit "The software update page shows “packagekit cannot refresh cache whilst offline” on a Debian or Ubuntu system."
https://cockpit-project.org/faq#error-message-about-being-offline

Create a placeholder file and network interface.
```bash
echo -e '[keyfile]\nunmanaged-devices=none' | sudo tee /etc/NetworkManager/conf.d/10-globally-managed-devices.conf
```
Set up a “dummy” network interface:
```bash
sudo nmcli con add type dummy con-name fake ifname fake0 ip4 1.2.3.4/24 gw4 1.2.3.1 && sudo reboot
```