
# Please use this how-to to reformat the Reduxio 

Reduxio  non-SED / (Self-Encrypted Drives) reformat process

## Check Reduxio Version
Check if the machines has Reduxio 3.4 RD14 installed first. In this case you may not need to re-install the systems. To check existing systems versions do the following:

1.  Login to system console using root user and root password
2.  Execute
    
    ```bash
     cat /reduxio/conf/package.control
    ```
    
3.  Do this on both controllers

## Reimage controllers with version 3.4 RD14
If either controller doesn't have 3.4 RD14, or you need to reimage for another reason: 

1. Create Reduxio bootable USB from ISO 
	1. Download Reduxio 3.4 RD14 iso from [support.reduxio.com](http://support.reduxio.com/) website 
	2.  Get Rufus from [https://rufus.ie/](https://rufus.ie/) (or use [Ventoy](ventoy.net) cross-platform)
	3.  Burn ISO with Rufus in DD mode 
	4.  *You might create two 8Gb or 4Gb size USB3.0 disk-on-keys*
2.  Turn on Reduxio and connect micro HDMI 2 VGA cable + keyboard with disk-on-key inserted to Bottom controller, while the TOP controller is physically out.
3.  Wait for the installation to complete and controller to reboot
4.  Now do the same with TOP controller while BOTTOM controller is out
5.  Needless to say that after the installation is complete, you need to remove the disk-on-keys
6.  Once installation is finished, while both controllers are inserted; turn on the Reduxio again and connect monitor + keyboard to bottom controller (it is an Active Controller)
  
## Proceed with System Format
1.  Login with username: root and password: disc|dia
2.  Do service reduxio killall on both controllers, first bottom one, then top one, do it quickly, so the controllers won't kill each other head.
3. Format the Bottom controller as Active controller and use the serial number which you can find on TOP of the box with silver sticker which has the following format: af4032f00**1a2**000e, you will need to extract the **1a2** HEX number and convert it to decimal format:  
  ```bash
	$ RDXSER=$(( 0x1a2 ))''
	$ echo $RDXSER
	418
  ```  
  (*alternatively, you can use [binaryhexconverter.com](https://www.binaryhexconverter.com/hex-to-decimal-converter)*)
  In this case the HEX number **1a2** is converted to **418** (that’s the number you would need to use while formatting)
4. Format the BOTTOM controller: 
```bash
format-system -env prod --activateProductionFlags -s $RDXSER
```
5.  Don’t reboot yet!
6.  Format the Top controller as Passive (move the VGA and keyboard):  
	 *(note the **-p** option for Passive)*
```bash
format-system -p -env prod --activateProductionFlags -s $RDXSDR
```

7.  Connect the VGA + Keyboard to Bottom controller again
8.  Get the storsense keys from Active controller. Get a fresh FAT16 or FAT32 formatted disk-on-key and insert it to Bottom Active controller
	1.  As root user and password: disc|dia
	2.  Find the disk-on-key and his corresponding device id; lsscsi (it should appear - usually as /dev/sdac)
	3.  mkdir /mnt/keys
	4.  mount /dev/sdac1 /mnt/keys or mount /dev/sdac /mnt/keys
	5.  cp /reduxio/persist/sotrsense/keys/* /mnt/keys/
	6.  umount /mnt/keys
	7.  Send the keys to [support-team@reduxio.com](mailto:support-team@reduxio.com) and [daniel@reduxio.com](mailto:daniel@reduxio.com) specifying serial number we will register it on storsense
9.  Reboot active controller and then passive controller using connected keyboard and VGA monitor
10.  Wait the machines to come up with ActiveDualSemiStopState (you should see the message on VGA screen
11.  Power off Reduxio wait for it to properly shutdown
12.  Machine is ready for deployment
13.  After installing it onsite, please perform field upgrade using 4.0
14.  Enable storsense from CLI or GUI after installation is completed

## In case of SED (Self-Encrypting Drives)

If drive encryption was turned on, then a file containing serial:PSID must be created and passed as an argument to format-system. The serial number and PSID can be found printed on the drive label. Using a Android phone to take images and Google Lens for OCR is useful, however watch for S->$ and Q->O and zero->O. The printed serial number is often (but not always) truncated compared to what is read from the drive, and expected. Using the following, I was able to read the drive serial numbers, then match them in vi with the truncated versions from the label:

```bash
for drive in e f g h i j k l m n o p q r s t u v w x y z aa ab;
do udevadm info --query=all --name=/dev/sd${drive} | 
	grep SCSI_IDENT_SERIAL;
done | sed 's/^.*=//' | tee serials.tmp
```

Alternatives to gather long serial numbers for '''snspid.txt''' file:
```bash
lsscsi|grep ST200|awk '{print $6}' |
while read dev;
do 
  sginfo -a $dev|grep Serial;
done
# **OR**
lsscsi | awk '/ST2000/ {print $6};/S841E/{print $6}' |
	xargs -n1 sginfo -a|sed -n "s/Serial Number '\(.*\)'$/\1/p"
```

Example format with serial/PSID file: 
```bash
format-system -env prod --activateProductionFlags -s 399 --psid_file_path /home/rat/snpsid.txt
# and
format-system -p -env prod --activateProductionFlags -s 399 --psid_file_path /home/rat/snpsid.txt
```


Instead of trying to do the ```service reduxio killall``` immediately upon boot above, can disable before reboot and re-enable reduxio and after system-format.

```
```bash
# current rc.d reduxio links:
ls -l /etc/rc?.d/*reduxio*
lrwxrwxrwx 1 root root 25 Oct 15  2018 /etc/rc2.d/S99wrapper-reduxio -> ../init.d/wrapper-reduxio
lrwxrwxrwx 1 root root 25 Oct 15  2018 /etc/rc3.d/S99wrapper-reduxio -> ../init.d/wrapper-reduxio
lrwxrwxrwx 1 root root 25 Oct 15  2018 /etc/rc4.d/S99wrapper-reduxio -> ../init.d/wrapper-reduxio
lrwxrwxrwx 1 root root 25 Oct 15  2018 /etc/rc5.d/S99wrapper-reduxio -> ../init.d/wrapper-reduxio
# Disable on startup
update-rc.d -f wrapper-reduxio remove
# Add back
update-rc.d -f wrapper-reduxio start 99 2 3 4 5 .
```