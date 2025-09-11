# Разбор lvm

### Если есть снапшот
sudo umount /var/my_lvm/snapshot
sudo lvremove /dev/my_volume_group/my_snapshot

### Разбор lvm
- Remove all logic volume
sudo lvscan
sudo umount /dev/vg_storage/lv_smb
sudo lvremove /dev/vg_storage/lv_smb

- Remove all volume group
sudo vgscan
sudo vgremove vg_storage

- Remove all physical volume
sudo pvscan
sudo pvremove /dev/sdc

- CLEAR
sudo dd if=/dev/zero of=/dev/sdc bs=1M count=1