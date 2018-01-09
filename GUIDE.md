## Examples
### Customize a firmware

Tested on DGN2200v4 with firmware version 1.0.0.98  
(You need to use Linux)

1) Backup configurations  
In case you make a bad custon firmware and you have to unbrick your modem router, you can restore your configs.

2) Download the original firmware  
You can find the latest version of your modem on netgear website (https://www.netgear.com/support/download/).  
For mine I used this one http://www.downloads.netgear.com/files/GDC/DGN2200V4/DGN2200v4-V1.0.0.98_1.0.98.zip  

3) Flash the original firmware  
Skip this step if your modem router is already running with the firmware version you are going to customize.  

4) Extract file system and kernel from the original firmware  
`./ambitImageEditor.py split -i Original_FW.chk -d extract`  
This will create a folder named `extract` with `kernel` and `rootfs` files.  

Note: this firmware is using only "Kernel" part of the image to store the RootFS (with the kernel inside). 
Different firmware may be different!  

5) Remove vtoken  
`./vtoken.py remove -i extract/rootfs -o rootfs.clean`
(You can find this tool inside tools directory)  
(Annotate the Flash type!)  

6) Decompress file system  
`sudo binwalk -e extract/rootfs.clean`  
Use sudo or `/dev` will be empty and you will make a corrupted custom firmware!  

7) Edit files inside `extract/_rootfs.clean.extract/jffs2-root/fs_1/`  
You can edit `/etc/profile` to run code at startup.  

8) Compress file system  
`mkfs.jffs2 -b -p -n -e 16384 -r fs_1 -o rootfs.new.clean -N $(pwd)/nocomprlist`  
You need to use a modified versoin of mkfs.jffs2 that support -N flag (precompiled binary is provided in this repo).  
Run this command inside `extract/_rootfs.clean.extract/jffs2-root/`.  
You can find `nocomprlist` file in tools directory of this repo.  
(The erase block size is needed only for NAND Flash Type [step 5]: NAND16 = 16384, NAND128 = 131072).  

9) Add vtoken  
`./vtoken.py merge -i extract/rootfs -c rootfs.new.clean -o extract/rootfs.new`  

10) Rebuild image  
`./ambitImageEditor.py merge -i Original_FW.chk -o Custom_FW.chk -k extract/kernel -r extract/rootfs.new`  
This will create a new firmware with the custom file system..  

11) Flash custom firmware  
Upgrade the firmware using Custom_FW.chk from the web interface like an official firmware.  