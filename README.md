# integrity-linux-modify-keys
A guide on how to prune system's efi image by removing content storing obscure windows keys. This may be helpful when migrating to full-linux setups.

### Instructions

#### inspect keys loaded by kernel
```bash
sudo dmesg | grep integrity
# examples:
X.509 cert 'Microsoft Corporation UEFI CA XXXX'
X.509 cert 'Microsoft Windows Production PCA XXXX'
```

#### you will need 'efitools' package (if on pacman - on other package managers the name might be different)
#### you will also need to have disabled Secure Boot, Microsoft CA
#### you will also need to have enabled Expert Key Management and/or Custom Mode and/or Audit mode
```bash
# first backup all keys
sudo efi-readvar -v db -o db-backup.esl
sudo efi-readvar -v KEK -o KEK-backup.esl
sudo efi-readvar -v dbx -o dbx-backup.esl

sudo lsattr /sys/firmware/efi/efivars/db-*
sudo lsattr /sys/firmware/efi/efivars/KEK-*
sudo lsattr /sys/firmware/efi/efivars/dbx-*

# remove immutability
sudo chattr -i /sys/firmware/efi/efivars/db-*
sudo chattr -i /sys/firmware/efi/efivars/KEK-*
sudo chattr -i /sys/firmware/efi/efivars/dbx-*

# remove from highest to lowest (format: List idx | Signature idx)
sudo efi-updatevar -e -d 3-0 db
sudo efi-updatevar -e -d 2-0 db

sudo efi-updatevar -e -d 0-0 dbx
sudo efi-updatevar -e -d 1-0 KEK

# restore immutability
sudo chattr +i /sys/firmware/efi/efivars/db-*
sudo chattr +i /sys/firmware/efi/efivars/KEK-*
sudo chattr +i /sys/firmware/efi/efivars/dbx-*
```

#### if you revert to your usual BIOS settings for key mgmt, the keys might be re-added, so choose appropriately
