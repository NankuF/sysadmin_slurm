# install virtualbox and vagrant to ubuntu 24.04

## get correct mirror
```bash
sudo nano /etc/apt/sources.list.d/ubuntu.sources
```
```text
Types: deb
URIs: http://security.ubuntu.com/ubuntu
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://mirror.yandex.ru/ubuntu/
Suites: noble noble-updates noble-security noble-backports
Components: main restricted universe multiverse
Signed-by: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```
```bash
sudo apt update
```
## install virtualbox
use curl or wget.<br>
curl:<br>
-L - redirect<br>
-O - оставить название файла без изменений
```bash
curl -OL https://download.virtualbox.org/virtualbox/7.2.0/virtualbox-7.2_7.2.0-170228~Ubuntu~noble_amd64.deb &&\
dpkg -i virtualbox-7.2_7.2.0-170228~Ubuntu~noble_amd64.deb
```
## install vagrant
```bash
curl -OL https://hashicorp-releases.yandexcloud.net/vagrant/2.4.9/vagrant_2.4.9-1_amd64.deb &&\
dpkg -i vagrant_2.4.9-1_amd64.deb
```
## run vm
```bash
cd to <dir/Vargantfile>
echo '* 0.0.0.0/0 ::/0' | sudo tee -a /etc/vbox/networks.conf
vagrant up
```