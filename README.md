# Home Server on a Laptop

This is a way to run local services on something most people have lying around (a laptop). You can stream your own movies and TV shows with Jellyfin, back up your phone's photos, and even host a Minecraft server.

## Hardware (you do NOT need this specific hardware, it's just what I used)
- Laptop: Dell Latitude 3500
- RAM: 8 GB DDR4
- Storage: 750 GB hard drive, plus an added 256 GB SSD and a 1 TB external hard drive

## Software
- OS: Ubuntu Server with CasaOS installed
- Services: Jellyfin (optional: Immich for photo backup, Crafty for a Minecraft server)

## Setup steps
1. Make a bootable USB drive. Download the Ubuntu Server ISO file from https://ubuntu.com/download/server and flash it to the USB with balenaEtcher, which you can download at https://etcher.balena.io/ (EVERYTHING ON THE USB WILL BE DELETED).
2. Move the laptop to its final place, which should be by your router, and plug it into power and ethernet.
3. Plug the USB into the laptop, turn it on, and press the boot menu key to boot from the USB. It is F12 on most Dell laptops, but it can be F10 or another key on other brands. (You might have to disable Secure Boot, which you can do in the BIOS.)
4. In the Ubuntu configuration select your language (English for me) and hit enter, then select your keyboard layout and hit enter. For the installation type I used the base Ubuntu Server option, so hit enter again. In the network configuration you should see your ethernet connection, so hit enter again.
5. In the proxy configuration don't add anything, just hit enter. I used the default mirror address, so hit enter again. In the storage configuration leave everything as default but deselect "Set up this disk as an LVM group", then go to Done and hit enter.
6. In the storage overview, as long as everything looks good, hit enter. A confirm destruction box will pop up. Scroll down to Continue and hit enter (THIS WILL WIPE EVERYTHING OFF THE DRIVE).
7. On the profile configuration screen enter your name, a name for your server (I used "laptop"), a username, and a strong password. Write these down. Tab down to Done and hit enter.
8. Ubuntu Pro isn't needed, so hit continue. On the SSH configuration screen hit enter on "Install OpenSSH server" to select it, then go down to Done and hit enter.
9. On the featured server snaps screen don't select anything, just go all the way down to Done and hit enter. The install will take a few minutes.
10. When it finishes select "Reboot Now", pull out the USB drive when it tells you to, then hit enter.
11. When it boots you will see the login screen. Enter your username and password (the password won't show as you type). Write down the IPv4 address in the middle of the screen (it looks like 192.168.x.x), you need it later.
12. (LAPTOPS ONLY) To keep it running with the lid closed, type `sudo nano /etc/systemd/logind.conf` and enter your password. Arrow down and delete the # in front of HandleLidSwitch, HandleLidSwitchExternalPower, HandleLidSwitchDocked, and LidSwitchIgnoreInhibited. Change the two that say =suspend to =ignore and change the =yes to =no. Hit Ctrl+X, then Y to save, then enter to confirm the file name. Then type `sudo reboot`.
13. Close the lid and put the laptop where it will live (don't block the air flow).
14. On your main PC open a terminal or command prompt and connect with `ssh YOUR-USERNAME@YOUR-SERVER-IP`. Type yes when it asks if you're sure, then enter your password. Anything you type here now runs on the server.
15. Update the server with `sudo apt update`, then `sudo apt upgrade` (type y to continue), then `sudo reboot`. Wait about 2 minutes and SSH back in with the same command as step 14.
16. Install CasaOS. Go to the CasaOS website, copy the install command (`curl -fsSL https://get.casaos.io | sudo bash`), paste it into your SSH window, and enter your password. It takes about 2 minutes.
17. On your main PC open a browser and go to your server's IP address. Click Go, then make a CasaOS username and password. You are now in the CasaOS dashboard.
18. (OPTIONAL) File sharing: open Files in CasaOS, right-click to make a new folder, click the three dots on it and hit Share. To use it from another PC, right-click the folder, choose "Get network path", paste it into your file explorer address bar, and log in with your CasaOS username and password. Your media folders (Movies, Music, TV Shows) are under Data > Media. You can share those the same way and copy your own media into them.

## Jellyfin (media server)
Jellyfin is a free, open-source media server (similar to Plex). It streams your movies and TV shows to any device on your network.

1. Install Jellyfin: open the CasaOS App Store, find Jellyfin, and click install. When it's done click to open it.
2. Set up Jellyfin: pick your language, then make a username and password. To add your libraries click the plus button, pick the content type (Movies), name it, click Add under Folders, and select Media > Movies. Do the same for TV Shows. Leave the rest as default, hit next through metadata language and remote access, then sign in.
3. Turn on hardware acceleration (needs an Intel CPU 7th gen or newer): in Jellyfin click the profile icon, go to Dashboard > Playback > Transcoding, set Hardware acceleration to Intel QuickSync, check the formats your media uses (like HEVC), scroll to the bottom and hit Save. Without this, playback was choppy and the CPU sat at 100%. With it, the CPU dropped to about 45%.

## Optional: Mount an external hard drive
(THIS ASSUMES YOU ARE OK WITH WIPING THE DRIVE. If it already has files you want to keep, skip steps 3 and 4 and see the note at the bottom.)

1. Plug the drive into the laptop and SSH into your server.
2. Find the drive by typing `lsblk`. Match it by size (mine is 1 TB). It will look something like `sdb` with a partition under it like `sdb1`. Your internal drive is usually `sda`. BE CAREFUL to pick the right one, the next steps erase it.
3. Create a partition on it. Type `sudo fdisk /dev/sdb` (use your drive name, not the partition). Press `g` to make a new GPT table, `n` to make a new partition (hit enter through the defaults), then `w` to save and exit.
4. Format the partition with `sudo mkfs.ext4 /dev/sdb1` (THIS ERASES EVERYTHING ON THE DRIVE).
5. Make a folder to mount it to: `sudo mkdir /mnt/external`
6. Find the drive's UUID by typing `sudo blkid /dev/sdb1` and copy the string after UUID= (without the quotes). The UUID stays the same even if the sdb name changes.
7. Open the mount config with `sudo nano /etc/fstab` and add this line at the bottom, using your UUID: `UUID=YOUR-UUID-HERE /mnt/external ext4 defaults,nofail 0 2`. The `nofail` part lets the server still boot if the drive is unplugged. Hit Ctrl+X, then Y, then enter to save.
8. Test it with `sudo mount -a`. If there are no errors, check that it worked with `df -h` and look for `/mnt/external` in the list. Check the fstab file carefully before you reboot, because a typo in it can cause boot problems.
9. Give yourself access to the drive with `sudo chown -R $USER:$USER /mnt/external`
10. To use the drive in Jellyfin: Jellyfin runs in a container, so it can only see folders you give it. In CasaOS open Jellyfin's settings, add a volume that maps `/mnt/external` to a folder inside the container, save, then add that folder as a library in Jellyfin. (The menu wording may differ slightly depending on your CasaOS version.)

NOTE: If your drive is already formatted as NTFS (a Windows drive) and has files you want to keep, skip steps 3 and 4. Use `sudo blkid` to get its UUID and use `ntfs-3g` instead of `ext4` in the fstab line. ext4 is better for a Linux server if you don't need to plug the drive into a Windows PC.

## Optional: Immich (phone photo backup)
1. In the CasaOS App Store install Immich. (The main one didn't work in the video, but the version without machine learning did.) Open it and click Get Started.
2. Enter an email, an admin password, and a name, then sign up and log in. Pick your theme and language and leave the privacy, storage, and backup settings as default.
3. Install the Immich app on your phone. For the server address enter `http://YOUR-SERVER-IP:2283`, then log in.
4. Select the photos or albums to back up and turn on backup. Your phone needs to be on the same network as the server.

## Optional: Minecraft server
1. In the CasaOS App Store install Crafty and open it. Ignore the "your connection is not private" warning by clicking Advanced, then Proceed.
2. Log in with the username `admin`. The password is in CasaOS Files under AppData > crafty > config > default-creds. Copy and paste it.
3. Click Create Server, choose Minecraft, then pick a server type (I used Fabric) and a version. Name it, set the RAM (min 2, max 4 in the video), leave the port at 25565, and hit Build Server.
4. Press the play button and agree to the Minecraft EULA.
5. In Minecraft go to Multiplayer > Direct Connect and enter `YOUR-SERVER-IP:25565`.
NOTE: This only covers connecting from devices on your home network. Letting friends join from outside your network needs extra setup (port forwarding or a tunneling service) that isn't covered here. Be careful with port forwarding, because it exposes the server to the internet.

## Credits
Used this video: https://www.youtube.com/watch?v=46T4cDQBkDs



