# Qemu setup instruction
1.	Introduction 
QEMU is free Arm emulator allows running Arm compiled program on Windows/Linux OS instead of on a target (for example Achilles board). 
It is usable when a development does not have a HW board to debug its code. 
Note: the QEMU does not emulate FPGA HW, thus it cannot be tested. This set (Image/Exe/Setup steps) allows setting and using Arm A7, Cortex-A9 emulation. 
It was created by Mayrom Rabinovich and re-tested (with little more explanation) by Yakow Erlich on Windows 10.

2.	 How to setup QEMU for Cortex-A9 with A7 ARM


2.1.	Install QUEMU on Linux working environment

This chapter depicts install on Linus virtual or non-virtual machines, on/off line.
2.1.1.	QEMU off-line Setup on Linux working environment
2.1.1.1.	Install QEMU packages
We are going to use QEMU over a Linux environment. Currently we are using Ubuntu 20.04 for the examples here.
For that, we will need to install the following packages: 
On Ubuntu use 
   sudo apt install <PACKAGE_NAME>
1.	qemu 
2.	qemu-user
3.	qemu-utils
4.	qemu-arm-static
5.	qemu-system-arm
6.	qemu-efi-arm
7.	qemu-kvm
(Some of them might be redundant).

2.1.1.2.	Run QEMU application

After installing the entire above packages on your machine via your package manager you should be able to run the command:
	qemu-system-arm [see full command line below]
If you are running the emulator on a Linux machine connected to Internet (virtual or not), you can skip to section ‎2.3 Running QEMU on page 4.

2.1.2.	QEMU off-line Setup a Linux working environment
If you are running the emulator on an offline Linux machine, you will need to install the above packages on that machine as well.
On Ubuntu, you can do it by fowling those steps:

1.	You must have online and offline machines (virtual or not) is running the same version of Ubuntu (e.g. 10.10 12.04, etc.) 
and have the same machine architecture (eg. x84, x64).

2.	Online machine. Perform all from ‎2.1.1.1 Install QEMU packages above.

3.	Online machine. Use apt to download all of your installed packages to folder
 /var/cache/apt/archives 
 
By running: 

dpkg -l | grep "^ii"| awk ' {print $2} ' | xargs sudo apt-get -y --force-yes install --reinstall --download-only 

This is going to take some time and might install many redundant packages but it is okay. 
If you want to install other package that you are missing on your offline machine like vim (good text editor) and 
open-vm-tools-desktop (allow clip bored and drag and drop form inside a ghost machine on VMware).

4.	Online machine. Create a tar archive from 

/var/cache/apt/archives 

by running the following commands 

cd /var/cache/apt 

tar -zcvf <full_path_to_new_tar_file> archives/*

5.	Offline machine. Copy that tar file to your offline computer and extract the tar by running anywhere on your machine:

tar -zxvf <new_tar_file> 

6.	Install all the .deb files inside of the archives directory that you just extracted by running: 
sudo dpkg -i ./archives/*deb

7.	Run QEMU as in ‎2.1.1.2 Run QEMU application above.

That is all, you are have QEMU on your offline machine! You can go to the next chapter.

2.2.	Setup a Windows working environment

For working with QEMU on Windows, we need to install QEMU with the qemu-installer. 
We use 
qemu-w64-setup-20200814.exe 
install file.
To run the install: Right click on install file / Run As Administrator 
After installing the qemu-installer, you will need to add the installer path (e.g. C:\Program Files\qemu) to your PATH environment variable as below:

•	My Computer/Properties/Advanced System settings

o	Environment Variables

o	Choose Path and click Edit

	Press New and enter the installer path in the new field and click OK


2.3.	Running QEMU
After completing the installation stage, you can start setting up your machine.

1.	First of all you will need to download the fowling:

•	An arm compiled Linux kernel

•	Cortex-A9 tree blob (Arm HW configuration).

•	Linux file system image

All of those files are stored inside arm3.tar.gz (for extraction use 7z application)

This tar is decompress to around 8.5 GB of data!

Make sure you have that kind of free disk space.

2.	After decompressing that tar file you can execute the following command to run you emulated board:

2.1.	On Linux :

qemu-system-arm -m 1024M -sd armdisk.img \

        -dtb vexpress-v2p-ca9.dtb \
        
        -M vexpress-a9 -cpu cortex-a9 \
        
        -kernel zImage \
        
        -append "root=/dev/mmcblk0p2" \
        
        -net nic -net user,hostfwd=tcp::2222-:22 \
        
        -no-reboot
        
2.2.	On Windows:

•	Open Windows PowerShell
•	cd < extracted arm3.tar.gz >
•	run the command below

qemu-system-arm -m 1024M -sd armdisk.img -dtb vexpress-v2p-ca9.dtb -M vexpress-a9 -cpu cortex-a9 -kernel zImage -append "root=/dev/mmcblk0p2" -net nic -net user,hostfwd=tcp::2222-:22,hostfwd=tcp::8888-:80 -no-reboot

Note: ignore the following warning message:

WARNING: Image format was not specified for 'armdisk.img' and probing guessed raw…

3.	After that, a black screen will show up and the machine will start to boot.

•	The user name of that machine is: user 

•	Password is: user

•	To become sudo you can run su and then enter the root password: root.


4.	Define foder that Eclipse IDE uses for application file transfer (default is /home/linaro):

su root

mkdir /home/linaro

5.	Authorize root access for SSH by editing SSHD sshd_config configuration file :

•	sudo vi /etc/ssh/sshd_config

•	then add the new line at the end:

•	PermitRootLogin Yes

•	Save the file

•	Reboot the Qemu


2.4.	QEMU port forwarding explanation

1.	Eclipse program (FW) and QEMU Arm emulator runs on the same computer. When Eclipse’s Debug tries to run SSH or SCP or other 
remote debugging programs, it initiates connecion to ports 22, 80, 5000 by the way like: [cmd params:localhost:port 22]  
(22 – ssh, 80 – http, 5000 – gdb-server). But how we direct these connections to ARM emulator, running on the PC with the same IP address
– localhost. The answer is to rename the ports 22, 80, 5000 in the ARM to another values X,Y,Z, to allow connect localhost:port Y. 
Such renaming is named as port forwarding procedure.

By port forwarding we can access localhost with the HOST_PORT and we will end up accessing the guest on the GUEST_PORT. 
It is done by adding hostfwd details in Qemu run command line
qemu-system-arm …

Details. The forwarding keywords and arguments:

hostfwd=<tcp/udp>::<HOST_PORT>-:<GUEST_PORT> 

to the arguments line
 -net nic -net user,... 
For example to port forward from 22 to 2222 and 80 to 8888, you need to run (Linux):

qemu-system-arm -m 1024M -sd armdisk.img \
      -dtb vexpress-v2p-ca9.dtb \
      -M vexpress-a9 -cpu cortex-a9 \
      -kernel zImage \
      -append "root=/dev/mmcblk0p2" \
      -net nic -net user,hostfwd=tcp::2222-:22,hostfwd=tcp::8888-:80 \
-no-reboot

2.	Example of usage ssh and scp commands.

ssh user@localhost –p 2222

ssh root@localhost –p 2222

scp -P 2222 WBRserver.axf user@localhost:/home/linaro

scp -P 2222 WBRserver.axf root@localhost:/home/root

3.	For setup a gdb server port (Linux debugger) add the forwarding:
hostfwd=tcp::5000-:5000

