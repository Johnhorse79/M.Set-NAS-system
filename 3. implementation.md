Section 3 — Implementation
3.1 Hardware Assembly
Physical assembly began with the Dell Precision 3630 Tower using the official Dell service manual as the primary reference guide.
Drive Installation:
The two WD Red Pro 14TB drives were installed first using aftermarket 3.5" brackets to secure them in the available drive bays. Each drive received both a power connector from the PSU and a dedicated SATA data cable connected to the motherboard. The motherboard presented four SATA ports — one black port was already occupied by the factory-installed 500GB Samsung SSD, one tan/cream colored port of unknown controller type was intentionally skipped, and the two remaining black ports received the WD Red Pro drives. All three drives were connected exclusively to black SATA ports to ensure consistent controller behavior.
NIC Installation:
The TP-Link TX201 2.5GbE NIC was installed into an available PCIe slot after the drives were secured. The driver disk included with the card was not used — TrueNAS SCALE includes native Linux kernel support for the Realtek RTL8125 chipset and detects the card automatically.
Display Solution:
The Precision 3630 has no HDMI output — only DisplayPort. Since no monitor was available during initial setup a DisplayPort to HDMI adapter was purchased and the PC was connected to a TV for the duration of the installation process. After initial setup was complete the TV was no longer needed as all TrueNAS administration is handled through the browser-based web UI.
Boot Drive Decision:
During assembly the factory-installed 500GB Samsung SSD was initially removed with the intention of using a hard drive in its place. After learning that TrueNAS SCALE requires a dedicated boot drive separate from the storage drives the SSD was reinstalled in its original bay. The final drive layout was:
BayDriveRoleSSD BaySamsung 860 EVO 500GBTrueNAS OS boot driveHDD Bay 1WD Red Pro 14TB (sdc)ZFS data poolHDD Bay 2WD Red Pro 14TB (sdb — defective, pending replacement)ZFS data pool

3.2 TrueNAS SCALE Installation
Creating the Boot Installer:
TrueNAS SCALE 25.10 was downloaded from truenas.com as a .iso file. Rufus was used to write the .iso to a USB flash drive creating a bootable installer.
Installation Process:
The USB drive was inserted into the Precision 3630 and the system was powered on. The Dell boot menu was accessed by pressing F12 during POST. USB Storage Device was selected as the boot target. TrueNAS booted into the installer and the following selections were made:

Install/Upgrade selected
500GB Samsung SSD selected as the boot drive
Root password set
Installation completed and USB removed
System rebooted from SSD

Initial Network Configuration:
On first boot TrueNAS displayed the Console Setup Menu. The system obtained an IP address automatically via DHCP from the local router. The IP address was used to access the TrueNAS web UI from a laptop browser for the first time.
Static IP Configuration:
As the NAS was moved between locations during setup the DHCP-assigned IP address changed each time making the system unreachable. Once the NAS was placed at its permanent location a static IP was configured:
SettingValueIP Address192.168.1.196/24Gateway192.168.1.1Primary DNS8.8.8.8Secondary DNS8.8.4.4
The static IP was configured through the TrueNAS web UI under Network → Interfaces and confirmed persistent across reboots.

3.3 Storage Configuration
Drive Verification:
Before creating the storage pool all three drives were verified in TrueNAS under Storage → Disks:
DriveModelSizeRolesdaSamsung 860 EVO465GBBoot drivesdbWDC WD140EFFX12.73 TiBData pool (defective — pending replacement)sdcWDC WD141KFGX12.73 TiBData pool (healthy)
ZFS Pool Creation:
After extensive troubleshooting of a defective drive (documented in Section 4) a single drive stripe pool was created using the healthy sdc drive as a temporary measure while awaiting a replacement drive:

Pool name: datapool
Layout: Single disk (temporary)
Usable capacity: 12.59 TiB
Status: Online, no errors

The pool will be converted to a ZFS mirror once the replacement drive arrives by adding the new drive and selecting mirror as the vdev type — ZFS will automatically sync all data to the new drive.
Dataset Creation:
Five datasets were created inside datapool to organize storage by user:
DatasetPathQuotaM.Setdatapool/M.Set10GB — shared family storagePeriondatapool/Perion100GBJoydatapool/Joy100GBSunshinedatapool/Sunshine100GBZadatapool/Za100GB
User Account Creation:
Four local user accounts were created in TrueNAS under Credentials → Local Users — one per family member. Each account was configured with:

SMB access enabled
Home directory mapped to their respective dataset path
TrueNAS admin access disabled for security


3.4 Nextcloud Installation
Nextcloud was installed through the TrueNAS Apps catalog (version 34.0.1) after one failed installation attempt that required a complete reinstall with corrected storage settings.
Final working configuration:
SettingValueAdmin UserPrecisePTimezoneAmerica/New_YorkHost192.168.1.196:30027Data Directory/mnt/datapoolPHP Upload Limit10GBMax Execution Time3600 secondsPHP Memory Limit512MBAppData StorageixVolume (TrueNAS managed)User Data StorageHost Path — /mnt/datapoolPostgres StorageixVolume (TrueNAS managed)CPUs2Memory4096MBWebUI Port30027
User Account Setup:
Four Nextcloud user accounts were created — one per family member — with 100GB storage quotas each. Each account maps to the user's dataset inside datapool.
Phone Auto-Sync:
The Nextcloud mobile app was installed on family iPhones. Auto-upload was enabled for photos and videos — all camera roll content syncs automatically to each user's private storage folder on the NAS.
Trusted Domains Configuration:
Trusted domains were configured via command line using the occ tool to allow access from both the local network IP and the Tailscale IP:
bashsudo su
docker exec -u www-data [container_id] php occ config:system:set trusted_domains 1 --value=192.168.1.196
docker exec -u www-data [container_id] php occ config:system:set trusted_domains 2 --value=100.76.193.123
docker exec -u www-data [container_id] php occ config:system:set trusted_domains 3 --value=192.168.1.196:30027
docker exec -u www-data [container_id] php occ config:system:set trusted_domains 4 --value=100.76.193.123:30027

3.5 Tailscale Installation
Tailscale was installed through the TrueNAS Apps catalog to enable secure remote access from any location without port forwarding.
Configuration:
SettingValueHostnametruenasAuth KeyGenerated from Tailscale admin panelAuth OnceEnabledAccept DNSEnabledCPUs1Memory256MB
Result:
TrueNAS joined the Tailscale network and received a permanent Tailscale IP address (100.76.193.123). All family devices with Tailscale installed can now access Nextcloud from anywhere in the world at:
http://100.76.193.123:30027
Access summary:
LocationAddressHome networkhttp://192.168.1.196:30027Remote — anywherehttp://100.76.193.123:30027

3.6 Offsite Backup — rclone + Azure Blob Storage
Azure Setup:
An Azure Blob Storage account was created with the following configuration:
SettingValueStorage accountnasfamilybackupRegionEast USPerformanceStandardRedundancyLRSAccess tierCoolContainernasbackup
rclone Configuration:
rclone was configured on TrueNAS to connect to Azure Blob Storage using the storage account name and access key. The remote was named azurebackup.
First Backup:
The initial backup was run manually as root with the truenas_admin config file specified explicitly:
bashsudo su
rclone copy /mnt/datapool azurebackup:nasbackup --progress --config /home/truenas_admin/.config/rclone/rclone.conf
The backup successfully transferred all datapool contents including Nextcloud application files and user data to Azure Blob Storage.
