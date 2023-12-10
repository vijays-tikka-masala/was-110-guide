# Bell Gigahub XGS-PON Bypass w/ WAS-

<!-- Table of Contents -->

# Table of Contents

- [Disclaimer: Liability and No-Responsibility Notice](#disclaimer-liability-and-not-responsibility-notice)
- [Credits](#credits)
- [Manufacturers](#manufacturers)
- [Gathering Gigahub Modem Attributes (DM#, SMB#, SGC#)](#gathering-gigahub-modem-attributes-dm-smb-sgc)
- [Connecting the WAS-](#connecting-the-was-)
- [Accessing the WAS-](#accessing-the-was-)
- [Checking for WAS- Issues](#checking-for-was--issues)
- [Setting WAS- Firmware Variables](#setting-was--firmware-variables)
- [Upgrading the WAS- to Custom Firmware](#upgrading-the-was--to-custom-firmware)
- [Fibre Connectivity to WAS-](#fibre-connectivity-to-was-)
- [PPPoE via the WAS-](#pppoe-via-the-was-)
- [Frequently Asked Question (FAQ)](#frequently-asked-question-faq)
  - [Internet VLAN - Untagged vs Tagged VLAN](#internet-vlan---untagged-vs-tagged-vlan)
  - [No WAS- Link to Switch with Custom Firmware and Fibre Disconnected](#no-was--link-to-switch-with-custom-firmware-and-fibre-disconnected)
  - [Downloading the Custom Firmware](#downloading-the-custom-firmware)
  - [WAS- Group Buy](#was--group-buy)

# Disclaimer: Liability and Not-Responsibility Notice

The information provided in this guide is intended for educational and informational purposes only. Users are solely responsible for the
application and implementation of the steps outlined in the guide. The authors, contributors, and distributors of this guide, including but not
limited to those credited, manufacturers, and the 8311 Discord Community, hereby declare that they are not liable for any direct, indirect,
incidental, consequential, or special damages, losses, or expenses arising from the use or misuse of the information provided.

The guide includes details about modifying hardware, upgrading firmware, and configuring network settings, which may involve risks and
potential hazards. Users are strongly advised to exercise caution, adhere to safety guidelines, and seek professional assistance if needed.
The authors do not guarantee the accuracy, completeness, or suitability of the information provided, and users acknowledge that they are
using the guide at their own risk.

Furthermore, the authors and contributors expressly disclaim any responsibility for the consequences of actions taken based on the
information presented in this guide. Users are encouraged to seek assistance from the 8311 Discord Community or other relevant support
channels if they encounter issues during the process.

The mention of specific individuals, manufacturers, or entities in the credits and acknowledgments section does not imply endorsement or
warranty of their products, services, or contributions. The guide is provided "as is," and no warranties, either express or implied, are made
regarding its contents.

By proceeding with the use of this guide, users acknowledge and agree to release the authors, contributors, and distributors from any
liability, claims, or damages that may arise in connection with the use of the information provided.

# Credits

None of this would’ve been possible if it wasn’t for the hard-working individuals over at the 8311 Discord Community ([link](https://discord.com/servers/8311-886329492438671420)).

Special thanks:

- up-n-atom ([Github](https://github.com/up-n-atom))
  - Original author for the steps ([8311 Discord Canada → FAQ](https://discord.com/channels/886329492438671420/1034609993451847680))
  - Original author for the Sagemcom XOM API CLI
- djGrrr ([Github](https://github.com/djGrrr))
  - Original author for the WAS-110 custom firmware
  - The latest custom firmware can be found over at the 8311 Discord Community ([link](https://discord.com/channels/886329492438671420/1162279893388759122/1178570504496496692))
- Miguel R.
  - Trusted distributor of the WAS-110 module

# Manufacturers

The WAS-110 is an Azores XGS-PON ONT SFP module ([link](https://azoresnetworks.com/product/pon-cpe-65.html)). Some manufacturers have rebadged the Azores XGS-PON modules, such
as:

- ECIN EN-XGSFPP-OMAC-V2 ([link](https://ecin.ca/custom-xgs-pon-sfp-stick-module-xgspon-ont-w-t-mac-function-mounted-on-sfp-package/))

# Gathering Gigahub Modem Attributes (DM#, SMB#, SGC#)

1. Grab your device DM## and SMB## values from the back of your Gigahub modem:

<a href="./doc-assets/images/gigahub-modem-back-shot.png" target="_blank"><img src="./doc-assets/images/gigahub-modem-back-shot.png" alt="Gigahub Modem Back Shot" width="25%"/></a>

2. Identify the firmware version your Gigahub is currently running
   a. Log into your Gigahub and find your Firmware version (accessing your Gigahub Web UI):
   (Image Credit: districtdogz)

```
b. Match your firmware version to the relative SGC# in the 8311 Discord channel (link)
```

```
i. Known values at time of writing (credit to up-n-atom):
```

1. **Home Hub 4000**
   **1.7.2** = SGC821011A
   **1.7.8.1** = SGC
   **1.7.11** = SGC
2. **Giga Hub**
   **1.16.3** = SGC830006E
   **1.16.5** = SGC830007C
   **1.19.5.1** = SGC83000C
   **1.19.5.4** = SGC83000D
   **1.19.6** = SGC83000DC

# Connecting the WAS-

## SFP Switch Method

Accessing the WAS-110 via a switch is possible if you have a 10G switch ready to go.

Some switches require you to have the fibre connected to establish a link. After you have the custom firmware, you can setup the WAS-
to link without fibre connected (see FAQ).

## Media Converter Method

If you don’t have a switch or the switch is not linking with your WAS-110, a media converter will work instead.

# Accessing the WAS-

1. Confirm you can access the WAS-110 by pinging it from your client device/virtual machine:
2. Go to [http://192.168.11.1,](http://192.168.11.1,) login with admin and password QsCg@7249#5281:
3. Go to the Service tab and tick the box on SSH

4. Start up a terminal and SSH into the WAS-110 with login root and password QpZm@4246#
   a. Linux
   ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.11.
   b. Windows 11
   ssh root@192.168.11.

```
c. If you had logged into the stick and it rebooted on you, you’ll have to clear your known_hosts file otherwise you’ll get an error since
the WAS-110’s SSH RSA fingerprint key changes after every reboot.
```

# Checking for WAS-110 issues

Check for issues on the WAS-110 by running the command below and ensuring that it returns nothing:

```
VOLS="kernelA bootcoreA rootfsA kernelB bootcoreB rootfsB rootfs_data ptconf" ; i=0; for VOL in $VOLS; do
VOLID=$(ubinfo /dev/ubi0 -N "$VOL" 2>/dev/null | grep 'Volume ID:' | awk '{print $3}'); [ -z "$VOLID" ] && echo
"Volume $VOL missing" || [ "$VOLID" -eq "$i" ] 2>/dev/null || echo "Volume $VOL misplaced (should be ID $i, not
$VOLID)"; i=$((i+1)); done
```

If you run into issues, seek support from the 8311 Discord community (link).

# Setting WAS-110 Firmware Variables

1. Ensure no issues are coming up with the WAS-110 (see here)
2. Issue the following commands while SSHed into the WAS-110 (replace your DM## , SMB##, and SGC## where applicable:

```
Note: Duplication of the commands are intentional
```

# Upgrading the WAS-110 to Custom Firmware

1. Ensure no issues are coming up with the WAS-110 (see here)
2. Download the latest firmware from the 8311 Discord Server (link)
   a. Extract local-upgrade.img from the archive file
3. On the WAS-110’s WEB UI:
   a. Upgrade to the custom firmware (note: you have to do this twice):
   i. Select browse and select the local-upgrade.img file
   ii. Select Upgrade

```
1 2 3 4 5 6 7 8 9
```

```
10
11
12
13
14
15
16
17
18
19
20
```

```
fw_setenv mib_file
fw_setenv mib_file
fw_setenv 8311_device_sn DM #############
fw_setenv 8311_device_sn DM #############
fw_setenv 8311_gpon_sn SMB #########
fw_setenv 8311_gpon_sn SMB #########
fw_setenv 8311_equipment_id 5690
fw_setenv 8311_equipment_id 5690
fw_setenv 8311_hw_ver Fast5689EBell
fw_setenv 8311_hw_ver Fast5689EBell
fw_setenv 8311_reg_id_hex 00
fw_setenv 8311_reg_id_hex 00
fw_setenv 8311_sw_verA SGC #######
fw_setenv 8311_sw_verA SGC #######
fw_setenv 8311_sw_verB SGC #######
fw_setenv 8311_sw_verB SGC #######
fw_setenv 8311_mib_file /etc/mibs/prx300_1V.ini
fw_setenv 8311_mib_file /etc/mibs/prx300_1V.ini
fw_setenv 8311_cp_hw_ver_sync 1
fw_setenv 8311_cp_hw_ver_sync 1
```

```
iii. Module will reboot
```

```
b. After the module comes back up, ensure there are no issues (see here)
c. Repeat the firmware upgrade from the previous step (yes, you must do this twice)
d. After the module comes back up a second time, ensure there are no issues (see here)
```

4. Leave the WAS-110 plugged in and ensure it stays up for 5 minutes without rebooting, you can spam pings to 192.168.11.1 to see if the
   device stays up.
5. If all is good, you can remove the failsafe and reboot the stick:

```
Note: If you don’t remove the failsafe, fibre won’t connect to the WAS-110. Ensure you remove the failsafe only if everything is good.
```

# Fibre Connectivity to WAS-

1. Plug the fibre into your WAS-
2. In the WAS-110’s Web UI at Status → PON, ensure it shows an ONU State of O5.
   If it’s not showing O5, your fibre isn't connected properly

```
1
2
```

```
rm -f /ptconf/.failsafe
reboot
```

# PPPoE via the WAS-

Using your choice of router (i.e. OPNSense, PFSense, Ubiquiti Dream Machine, etc.), setup PPPoE (b1id/password) like you normally
would’ve done with the Gigahub.

# Frequently Asked Question (FAQ)

## Internet VLAN - Untagged vs Tagged VLAN

The custom firmware will default to untagging the Internet VLAN for PPPoE.

If you still wish to have the Internet Service VLAN tagged to 35, check the djgrr’s docs on Github (link)

## No WAS-110 link to Switch with Custom Firmware and Fibre Disconnected

The WAS-110 asserts RX_LOS which some switches (e.g. Mikrotik) monitor to establish a link. You need to disable this.

1. Re-establish connectivity from the device that you previously SSHed the WAS-110 from
2. Issue the following SSH commands to update the 8311_rx_los variable to a value of 0
3. Reconnect the WAS-110 into the switch, you should now get a link even with the fibre disconnected

## Downloading the Custom Firmware

The latest custom firmware by djGrr for the WAS-110 can be obtained from the 8311 Discord Community (link).

## WAS-110 Group Buy

If you’re interested in obtaining one through a group buy, check out the 8311 Discord Buy and Sell channel (link).

```
1
2
```

```
fw_setenv 8311_rx_los 0
fw_setenv 8311_rx_los 0
```
