# HOW TO CONFIGURE CISCO MDS SCHEDULED BACKUPS 

In my example I will create a service user on the switch username `ansible`, Substitute `P@ssw0rd` for your password.  My ansible server is `ansible-control01.example.net` and `/data/backup` on this device is an NFS mount to a NAS filer.

## CREATE A NEW SERVICE USER ON THE SWITCH

`ssh admin@MDS-A9710.example.net`

```bash
MDS-A9710# config t
Enter configuration commands, one per line.  End with CNTL/Z.
MDS-A9710(config)# username ansible password P@ssw0rd123 role network-admin
MDS-A9710(config)# copy r s
[########################################] 100%
Copy complete.
```

## TO CREATE A NEW SSH KEY 

```bash
MDS-A9710(config)# username ansible keypair generate rsa 1024
generating rsa key(1024 bits).....
.
generated rsa key
MDS-A9710(config)# show username ansible keypair
**************************************

rsa Keys generated:Wed Mar  2 15:14:12 2022

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQCuD0r5gHIhiD7pqwer83RPAGYYa1YD57dxHAKZVDTKlG/NOTAREALSSHKEY//NZ1bGIzsm+UwV/0kXvA3lmefRKLVvQHm+Pat1EZUSY80ei5UjJAYFAI9EKudIPJsyS/OwGHBmphDfkrTQBdtj3B3T5XQKcS87dJ
5PNYakEY1w==

bitcount:262144
fingerprint:
MD5:77:23:6e:36:fk:b6:4c:90:d5:5c:fd:f8:ca:23:1d:48**************************************

could not retrieve dsa key information
bitcount:0
**************************************

could not retrieve ecdsa key information0
**************************************
MDS-A9710(config)# cop r s
[########################################] 100%
Copy complete.
MDS-A9710(config)# username ansible keypair export bootflash:ansible_rsa rsa
MDS-A9710(config)# dir bootflash:
       4096    Nov 17 07:41:36 2021  .patch/
          0    Feb 02 10:05:01 2022  20220202_170501_poap_5744_init.log
        887    Mar 02 15:15:47 2022  ansible_rsa
        232    Mar 02 15:15:48 2022  ansible_rsa.pub
      16384    Nov 17 07:35:51 2021  lost+found/
   69206528    Nov 17 07:37:17 2021  m9700-sf3ek9-kickstart-mz.8.3.2.bin
  418518384    Nov 17 07:38:04 2021  m9700-sf3ek9-mz.8.3.2.bin
       4096    Feb 14 13:37:17 2022  scripts/

Usage for bootflash://sup-local
  868319232 bytes used
 6577778688 bytes free
 7446097920 bytes total
MDS-A9710(config)# copy bootflash:ansible_rsa.pub sftp://ansible@ansible-control01.example.net/home/ansible/.ssh/$(SWITCHNAME)_ansible_rsa.pub
```

In another terminal window:
`ssh ansible@ansible-control01.example.net`

```bash
cd /home/ansible/.ssh/
cat $(SWITCHNAME)_ansible_rsa.pub >> authorized_keys2
```

SSH back to your MDS switch terminal window as ansbile user.
`ssh ansible@MDS-A9710.example.net`


## TO USE AN EXISTING SSH KEY 

```bash
MDS-A9710(config)# copy scp://ansible@ansible-control01.example.net/home/ansible/.ssh/ansible_rsa bootflash:ansible_rsa
MDS-A9710(config)# copy scp://ansible@ansible-control01.example.net/home/ansible/.ssh/ansible_rsa.pub bootflash:ansible_rsa.pub
MDS-A9710(config)# username ansible keypair import bootflash:ansible_rsa rsa
Enter Passphrase: *********** (P@ssw0rd123)
MDS-A9710(config)# copy r s
[########################################] 100%
MDS-A9710(config)# end
MDS-A9710# exit
```

SSH back to your MDS switch terminal window as ansbile user.
`ssh ansible@MDS-A9710.example.net`

```bash
MDS-A9710# config t
MDS-A9710(config)# copy running-config startup-config
MDS-A9710(config)# copy start sftp://ansible@ansible-control01.example.net/data/backup/$(SWITCHNAME)/$(SWITCHNAME)_$(TIMESTAMP).cfg
```

This should create a backup config file on `ansible-control01.example.net/data/backup/$(SWITCHNAME)/` without prompting you for a password!  if it fails ensure there is a folder named the same as the switch name in the /data/backup/ directory. 


## PROCEED AFTER SSH KEY IS IN PLACE

## CREATE BACKUP SCHEDULE

```bash
MDS-A9710(config)# scheduler enable
MDS-A9710(config)# scheduler job name backup_config
MDS-A9710(config-job)# copy running-config startup-config
MDS-A9710(config-job)# copy start sftp://ansible@ansible-control01.example.net/data/backup/$(SWITCHNAME)/$(SWITCHNAME)_$(TIMESTAMP).cfg
MDS-A9710(config-job)# exit
MDS-A9710(config)# show scheduler job name backup_config
Job Name: backup_config
-----------------------
copy running-config startup-config
 copy startup-config sftp://ansible@ansible-control01.example.net/data/backup/$(SWITCHNAME)/$(SWITCHNAME)_$(TIMESTAMP).cfg 
 
==============================================================================
MDS-A9710(config)# scheduler schedule name nightly_10pm
MDS-A9710(config-schedule)# time daily 22:00
MDS-A9710(config-schedule)# job name backup_config
MDS-A9710(config-schedule)# email-addr my_email@example.net
MDS-A9710(config-schedule)# exit
MDS-A9710(config)# show scheduler schedule name nightly_10pm
Schedule Name       : nightly_10pm
----------------------------------
User Name           : ansible
Schedule Type       : Run every day at 22 Hrs 0 Mins
Last Execution Time : Yet to be executed
-----------------------------------------------
     Job Name            Last Execution Status
-----------------------------------------------
backup_config                         -NA-
==============================================================================
MDS-A9710(config)# exit
MDS-A9710# copy r s
[########################################] 100%
Copy complete.
MDS-A9710# copy start sftp://ansible@ansible-control01.example.net/data/backup/$(SWITCHNAME)/$(SWITCHNAME)_$(TIMESTAMP).cfg
```

!!! The above command should complete without needing to enter a password.  If sucessful you are done.