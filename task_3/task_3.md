[newredos@mephi-2026 Desktop]$ lsblk -f
NAME FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                         
├─sda1
│                                                                           
├─sda2
│    ext4   1.0         a0c63fb2-26f5-4359-aa6e-2fabc273d1f3  688.1M    22% /boot
├─sda3
│    swap   1           9f61c21a-9492-4538-9743-bdf740a49a53                [SWAP]
└─sda4
     ext4   1.0         1361d9ff-a252-4511-a362-a49f5e80a1ca    7.9G    59% /
sdb                                                                         
└─sdb1
     ext4   1.0   MEPHI_WEB
                        26d9f23a-6569-4f1a-8406-26a519c106d5    1.8G     0% /mephi-web
sr0                                                                         
zram0



[newredos@mephi-2026 Desktop]$ df -h /mephi-web
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       2.0G   24K  1.9G   1% /mephi-web
