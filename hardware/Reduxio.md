
# Please use this how-to to reformat the Reduxio box
(non SED)
  

Reduxio (non Encrypted drives reformat process)

Check if the machines has Reduxio 3.4 RD14 installed first, in this case you don't need to re-install the systems, to check existing systems versions do the following:

1. Login to system console using root user and root password

2. do cat /reduxio/conf/package.control 

3. Do this on both controllers

4. If both controllers has 3.4 RD14 you can skip to section 10 in below how-to 

	1.  Get Reduxio 3.4 RD14 iso from [support.reduxio.com](http://support.reduxio.com/) website and download it to Windows machine
	2.  Get Rufus from [https://rufus.ie/](https://rufus.ie/)
	3.  Burn ISO with Rufus in DD mode 
	4.  You might create two 8Gb or 4Gb size USB3.0 disk-on-keys
	5.  Turn on Reduxio and connect micro HDMI 2 VGA cable + keyboard with disk-on-key inserted to Bottom controller, while the TOP controller is physically out.
	6.  Wait for the installation to complete and controller to reboot
	7.  Now do the same with TOP controller while BOTTOM controller is out
	8.  Needless to say that after the installation is complete, you need to remove the disk-on-keys
	9.  Once installation is finished, while both controllers are inserted; turn on the Reduxio again and connect monitor + keyboard to bottom controller (it is an Active Controller)
	10.  Login with username: root and password: disc|dia
	11.  Do service reduxio killall on both controllers, first bottom one, then top one, do it quickly, so the controllers won't kill each other head.
	12.  Format the Bottom controller as Active controller and use the serial number which you can find on TOP of the box with silver sticker which has the following format: af4032f00**1a2**000e, you will need to extract the **1a2** HEX number and convert it to decimal format using a calculator or this website: [https://www.binaryhexconverter.com/hex-to-decimal-converter](https://www.binaryhexconverter.com/hex-to-decimal-converter) in this case the HEX number **1a2** is converted to **418** (that’s the number you would need to use while formatting)
	13.  Format the BOTTOM controller: 
	    ```bash
format-system -env prod --activateProductionFlags -s 418
```
	14.  Don’t reboot yet
	15.  Format the Top controller as Passive (move the VGA and keyboard):  
   	     *(note the **-p** option for Passive)*
```bash
format-system -p -env prod --activateProductionFlags -s 418
  ```
	16.  Connect the VGA + Keyboard to Bottom controller again
	17.  Get the storsense keys from Active controller. Get a fresh FAT16 or FAT32 formatted disk-on-key and insert it to Bottom Active controller
		1.  As root user and password: disc|dia
		2.  Find the disk-on-key and his corresponding device id; lsscsi (it should appear - usually as /dev/sdac)
		3.  mkdir /mnt/keys
		4.  mount /dev/sdac1 /mnt/keys or mount /dev/sdac /mnt/keys
		5.  cp /reduxio/persist/sotrsense/keys/* /mnt/keys/
		6.  umount /mnt/keys
		7.  Send the keys to [support-team@reduxio.com](mailto:support-team@reduxio.com) and [daniel@reduxio.com](mailto:daniel@reduxio.com) specifying serial number we will register it on storsense
	18.  Reboot active controller and then passive controller using connected keyboard and VGA monitor
	19.  Wait the machines to come up with ActiveDualSemiStopState (you should see the message on VGA screen
	20.  Power off Reduxio wait for it to properly shutdown
	21.  Machine is ready for deployment
	22.  After installing it onsite, please perform field upgrade using 4.0
	23.  Enable storsense from CLI or GUI after installation is completed


If drive encryption was turned on, then a file containing serial:PSID must be created and passed as an argument to format-system. The serial number and PSID can be found printed on the drive label, however I found the serial number was often (but not always) truncated. Using the following, I was able to read the drive serial numbers, then match them in vi with the truncated versions from the label:

```bash
for drive in e f g h i j k l m n o p q r s t u v w x y z aa ab;
do udevadm info --query=all --name=/dev/sd${drive} | 
	grep SCSI_IDENT_SERIAL;
done | sed 's/^.*=//' | tee serials.tmp
```

Example format with serial/PSID file: 
```bash
format-system -env prod --activateProductionFlags -s 399 --psid_file_path /home/rat/snpsid.txt
```
