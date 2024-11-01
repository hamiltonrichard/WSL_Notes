# Recovering Data from an OpenSUSE Linux instance running in Windows Subsystem for Linux

I had a hardware failure which took down my Windows 11 machine. I needed to recover data from an instance of OpenSUSE running in the Windows Subsystem for Linux. These instructions should work for any Linux distribution with the libguestfs and ntfs-3g packages are installed. _libguestfs_ also uses _fuze_. For Debian based distributions, ensure your user is part of the "fuze" group.

## Preperation for Recovery

According to the information I found the recovery can be done using _libgestfs_. It creates a virtual machine to access the disk images. _libguestfs_ uses kvm and qemu components to create the virtual machine.

Since the filesystem on the M.2 MVME drive is NTFS, _ntfs-3g_ should be installed as well. 

Install the the package using:
```bash
sudo zypper install libguestfs ntfs-3g
```
Any dependancies for _libguestfs_ and _ntfs-3g_ will be installed as well. 

Note:

There is a dependancy on fuse. This is a core component of OpenSUSE and should be for others as well. Check your Distribution's documentation if you're not using OpenSUSE.

## Mounting the OpenSUSE Linux Instance


The WSL images are located in %LocalAppData%\Packages\ on the Windows drive (C:\Users\richh\AppData\Local\Packages).  There will be directories for each Linux instance. Each instance of Linux will have its on directory. The images themselves are *.vhdx files which are in the _LocalState_ subdirectory in Packages

### Mounting the Windows Filesystem

In my case the filesystem automounted on /run/media/.  You can simply use a _mount (8)_ command if needed. Once mounted, proceede to /Users/[USERNAME]/AppData/Local/Packages. 

You can use the find command to locate the *.vhdx file for the Linux instance. 

```bash
find ./ -name *.vhdx
```
In this instance I have not only an OpenSUSE instance, but an Ubuntu one as well
```
./46932SUSE.openSUSELeap15.6_022rs5jcyhyac/LocalState/ext4.vhdx
./CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc/LocalState/ext4.vhdx
```

### Copying the VHDX file to a New Location (Optional)

I didn't want to work on the origional VXDF file so I copied to a different location. This is optional. In my case I created a wsl and wsl_images directory in my home directory and copied the image to wsl_images:

```bash
mkdir wsl wsl_images/
cp /run/media/rhamilton/Windows-SSD/Users/richh/AppData/Local/Packages/46932SUSE.openSUSELeap15.6_022rs5jcyhyac/LocalState/ext4.vhdx ~/wsl_images/

```
This may take a few minutes to copy. My particular image was 62Gig. 

### Mounting the VHDX File

Using `guestmount` and `guestumount` you can mount and unmount the image


From the wsl_image directory I executed:

```bash
guestmount -i -a ./ext4.vhdx --ro ../wsl
```
The options break down as:

__-i__ Using virt-inspector(1) code, inspect the disks looking for an operating system and mount filesystems as they would be mounted on the real virtual machine.

__-a__ Using virt-inspector(1) code, inspect the disks looking for an operating system and mount filesystems as they would be mounted on the real virtual machine.

__--ro__ Add devices and mount everything read-only.  Also disallow writes and make the disk appear read-only to FUSE. 


You can also execute a mount command and grep for the mountpoint . 

```bash
mount | grep wsl
```
In my case this returned:

```
/dev/fuse on /home/rhamilton/wsl type fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=100)
```
Now that the image is mounted you cd change directries to the mounted image. Since I was in the _wsl_image_ directory I executed:

```bash
cd ../wsl/
``` 
### Unmounting the VHDX File

Once you are done working with the image simply unmount it using _guestunmount_

```bash
cd ..
guestunmount wsl/
```

## Manpage info for _guestmount_

Review _guestmount(1)_ for complete details. These notes may be of some interest:

* Debian by default doesn't add user to the fuse group by default. Check the Debian documentation for details. 
* If you need to allow access to the instance file system to others then add the -o allow_other parameter to _guestmount_. Ensure that the proper permissions to access the mount point are in place.  

## Sources

1. [Finding or Recovering WSL Data](https://christopherkibble.com/posts/wsl-vhdx-recovery/)
