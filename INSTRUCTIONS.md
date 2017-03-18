## Instructions for creating custom vm template in OSX

* Install xhyve if it is not installed yet:

	`$ brew install xhyve`

* Clone freenas vm-templates repository:

	`% git clone git@github.com:freenas/vm-templates.git`

* Create new folder for your template:

	`% mkdir mytempalte && cd mytemplate`
	
* Copy template.json from existing template that matches your os (in my case it was ubuntu 16.04 server):

	`% cp ../vm-templates/ubuntu-server-16.04-vnc/template.json .`

* Create os image file:

	`% mkfile 16g os.img`

* Copy initrd.gz and vmlinuz from installation iso install/ folder (I simply used [unarchiver](http://unarchiver.c3.cx/unarchiver) to extract **ubuntu-16.04.2-server-amd64.iso** contents):

	`% cp ../ubuntu-16.04.2-server-amd64/install/vmlinuz .`
	`% cp ../ubuntu-16.04.2-server-amd64/install/initrd.gz .`

* Create install script for xhyve:

```
#!/bin/sh

UUID=`uuidgen`

# make sure this path exists 
# (I used brew to install xhyve, so this was 
# path on my system, could be different on yours):
USERBOOT="/usr/local/share/xhyve/test/userboot.so"
BOOTVOLUME="ubuntu-16.04.2-server-amd64.iso"

# I tried downloading vmlinuz and initrd form ubuntu website, 
# but installer complained that it does not match one in iso

IMG="os.img"
KERNELENV=""

MEM="-m 2G"
SMP="-c 2"
PCI_DEV="-s 0:0,hostbridge -s 31,lpc"
NET="-s 2:0,virtio-net"
IMG_CD="-s 3:0,ahci-cd,$BOOTVOLUME"
IMG_HDD="-s 4:0,virtio-blk,$IMG"
LPC_DEV="-l com1,stdio"
ACPI="-A"
CMDLINE="earlyprintk=serial console=ttyS0"

xhyve $ACPI $MEM $SMP $PCI_DEV $LPC_DEV $NET $IMG_CD $IMG_HDD -U $UUID -f kexec,vmlinuz,initrd.gz,"$CMDLINE"
```

* Make it executable:
	
	`% chmod a+x ./install.sh`

* Start os installation:
	
	`% sudo ./install.sh`

* After install completes:

	`% gzip -9 os.img`

* You will need to set this shasum inside template.json:

	`% shasum -a 256 os.img.gz`
	
* Upload os.img.gz to Docker or some other service that you have access to and update image url inside template.json (*fetch* section)

### References:

* [Official FreeNAS templates](https://github.com/freenas/vm-templates)
* [Install Ubuntu on xhyve](https://github.com/LnL7/ubuntu-xhyve)
* [Install FreeBSD on xhyve](https://gist.github.com/tanb/f8fefa22332edc7a641d)
* [FreeNAS Wiki for creating VM template](https://wiki.freenas.org/index.php/Creating_your_own_VM_Template)