1. Install Ubuntu with LUKS enabled. Use Wayland sessions.
2. Select all privacy options from configuration.
3. Set updates server to central. And update system.
4. Install ufw + gufw and enable. Deny all incoming, Allow all outgoing. (we install OpenSnitch later.):
   _Install ufw_
`sudo apt-get update
sudo apt-get install ufw gufw
sudo ufw enable`
_TODO more_

5. Install VLC from Snap (enable permissions or deny as you see).
6. Some GNOME privacy configurations. (For each account)
#[PRIVACY] [HAS GUI] Disable nautilus thumbnails
dconf write /org/gnome/nautilus/preferences/show-image-thumbnails "'never'"

#[PRIVACY] [HAS GUI] Do not keep recent opened files list
dconf write /org/gnome/desktop/privacy/remember-recent-files false

#[PRIVACY] [HAS GUI] Remove old temporal files auto
dconf write /org/gnome/desktop/privacy/remove-old-temp-files true

#Changes that cannot be set via Gnome Settings GUI

#[PRIVACY] Disable global thumbnails
dconf write /org/gnome/desktop/thumbnailers/disable-all true

#[HARDENING] [Still no GUI] Enable USB attack protection on lockscreen
dconf write /org/gnome/desktop/privacy/usb-protection true
dconf write /org/gnome/desktop/privacy/usb-protection-level "'lockscreen'"


#Install apparmor-profiles and apparmor-profiles-extra
sudo apt-get install apparmor-profiles apparmor-profiles-extra


#Use checksec.sh and enable the disabled options.
#Use flatpacks or snaps.

#Private Internet Access: if client GUI is killed, remain connected on the background.
#The process is run as root but piactl can be changed with our current user, this is a vulnerability.
piactl background enable

sudo chmod -x /opt/piavpn/bin/piactl


#Investigate how to disable gvfs-metadata
# --disable-gvfs-metadata
#https://gitlab.gnome.org/GNOME/gvfs/-/issues


#Kernel disable unprivileged userns!
sysctl kernel.unprivileged_userns_clone=0
#TODO: see how to make it permanent

-> Remove user from Login screen: (only that user, not all users, is better to hide things)
https://www.thegeekdiary.com/how-to-disable-user-list-on-gnome-login-screen-in-centos-rhel-8/

7. Create a new user and enable eCryptFS (until new crypto is ok):
   https://www.linuxuprising.com/2018/04/how-to-encrypt-home-folder-in-ubuntu.html

-> FIX Logout does not umount eCryptFS:

version 1)
https://askubuntu.com/questions/910484/encrypted-home-folder-still-accessible-after-logout

If you do not need access from cron or at jobs (non-interactive tasks) to ANY home directories then you just need to comment out the line

session  optional        pam_ecryptfs.so unwrap

in /etc/pam.d/common-session-noninteractive.

This will cause all encrypted home directories to be unmounted when the user logs out.

-> Install usbkill: (and at the same time gnome usn protection enabled)
https://github.com/hephaest0s/usbkill

-> Make a panic password for the main account, to execute actions in the background. (Warning: not compatible with eCryptFS. You cannot use it on accounts with eCryptFS, at least with the allow option)
apt-get install git gcc make libpam0g-dev libssl-dev
https://github.com/hellresistor/pam_duress
pam configuration working on ubuntu 21.04:
```
# here are the per-package modules (the "Primary" block)
auth	[success=3 default=ignore]	pam_unix.so nullok_secure
auth	[success=2 default=ignore]	pam_sss.so use_first_pass
auth    [success=1 default=ignore]      pam_duress.so allow
# here's the fallback if no module succeeds
auth	requisite			pam_deny.so
# prime the stack with a positive return value if there isn't one already;
# this avoids us returning an error just because nothing sets a success code
# since the modules above will each just jump around
auth	required			pam_permit.so
# and here are more per-package modules (the "Additional" block)
auth	required	pam_ecryptfs.so unwrap
auth	optional			pam_cap.so
# end of pam-auth-update config
```


-> Opensnitch:
https://github.com/evilsocket/opensnitch

-> Kernel hardening: 5.11.

1. Scan with checksec script: https://github.com/slimm609/checksec.sh
   

