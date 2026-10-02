\# Oracle Database 19c Lab VM



This project creates a ready-to-use Oracle Linux 9 virtual machine using Vagrant and VMware Workstation for Oracle Database 19c DBA practice.



The goal is to automate the operating system setup, Oracle prerequisites, filesystem preparation, swap configuration, and Oracle environment so the VM is ready for Oracle Database 19c installation.



\---



\## Lab Architecture



Windows Host

&#x20;   |

&#x20;   v

Vagrant

&#x20;   |

&#x20;   v

VMware Workstation

&#x20;   |

&#x20;   v

Oracle Linux 9 VM

&#x20;   |

&#x20;   +-- 8 GB RAM

&#x20;   +-- 4 CPUs

&#x20;   +-- 8 GB Swap

&#x20;   |

&#x20;   +-- /u01 100 GB

&#x20;         |

&#x20;         +-- /u01/app/oracle

&#x20;         +-- /u01/app/oraInventory

&#x20;         +-- /u01/software

&#x20;         +-- /u01/oradata

&#x20;         +-- /u01/fast\_recovery\_area



\---



\## Lab Configuration



| Component | Configuration |

|---|---|

| Operating System | Oracle Linux 9 |

| Virtualization | VMware Workstation |

| Provisioning | Vagrant |

| Hostname | oracle19c-vm |

| Private IP | 192.168.136.20 |

| RAM | 8 GB |

| CPU | 4 |

| Swap | 8 GB |

| Oracle Filesystem | /u01 |

| /u01 Size | 100 GB |

| Oracle Database Version | Oracle Database 19c |



\---



\## Oracle Environment



ORACLE\_BASE=/u01/app/oracle

ORACLE\_HOME=/u01/app/oracle/product/19.0.0/dbhome\_1

ORACLE\_SID=ORCL



\---



\## Filesystem Layout



/u01

├── app

│   ├── oracle

│   │   └── product

│   │       └── 19.0.0

│   │           └── dbhome\_1

│   └── oraInventory

├── software

├── oradata

└── fast\_recovery\_area



Directory Purpose:



/u01/app/oracle

&#x20;   Oracle Base



/u01/app/oracle/product/19.0.0/dbhome\_1

&#x20;   Oracle Home



/u01/app/oraInventory

&#x20;   Oracle Inventory



/u01/software

&#x20;   Oracle installation software and patches



/u01/oradata

&#x20;   Database data files



/u01/fast\_recovery\_area

&#x20;   FRA / recovery files



\---



\## What the Vagrantfile Does



Running:



vagrant up



automatically performs the following tasks:



\- Creates an Oracle Linux 9 VM

\- Configures 8 GB RAM

\- Configures 4 CPUs

\- Configures hostname oracle19c-vm

\- Configures private IP 192.168.136.20

\- Enables SSH password authentication

\- Creates a 100 GB XFS filesystem mounted on /u01

\- Adds /u01 to /etc/fstab

\- Creates an 8 GB swap file

\- Makes swap persistent using /etc/fstab

\- Installs oracle-database-preinstall-19c

\- Creates the Oracle OS user and required groups

\- Configures Oracle kernel parameters

\- Configures Oracle user limits

\- Creates Oracle directory structure

\- Configures Oracle environment variables



\---



\## Prerequisites



Install the following software on the Windows host:



\- VMware Workstation

\- Vagrant

\- Vagrant VMware Desktop plugin

\- Git

\- MobaXterm or another X11-capable SSH client for Oracle GUI installation



Check Vagrant:



vagrant --version



Check installed plugins:



vagrant plugin list



You should see:



vagrant-vmware-desktop



\---



\## Create the VM



Clone the repository:



git clone https://github.com/arunshinde89/Oracle-19c-vm.git



Go to the project directory:



cd Oracle-19c-vm



Create and provision the VM:



vagrant up



The first build can take several minutes because Oracle Linux packages and prerequisites need to be downloaded and installed.



\---



\## Check VM Status



vagrant status



Expected result:



default    running (vmware\_desktop)



\---



\## Connect to the VM



Using Vagrant:



vagrant ssh



Or use PuTTY / MobaXterm.



Host     : 192.168.136.20

Port     : 22

User     : oracle



\---



\## Verify /u01



Run:



df -hT /u01



Expected result should show approximately:



Filesystem    Type   Size   Mounted on

/dev/loop0    xfs    100G   /u01



The filesystem is backed by a sparse image file:



/var/lib/oracledisks/u01.img



The /etc/fstab entry is:



/var/lib/oracledisks/u01.img /u01 xfs loop,nofail 0 0



\---



\## Verify Swap



Run:



free -h



or:



swapon --show



Expected swap size:



8 GB



The swap file is:



/swapfile



and is configured persistently in /etc/fstab.



\---



\## Verify Oracle User



Run:



id oracle



The Oracle preinstall package creates the required Oracle user and operating system groups.



\---



\## Verify Oracle Environment



Switch to the Oracle user:



su - oracle



Check:



echo $ORACLE\_BASE

echo $ORACLE\_HOME

echo $ORACLE\_SID



Expected output:



/u01/app/oracle

/u01/app/oracle/product/19.0.0/dbhome\_1

ORCL



\---



\## Oracle 19c Software



Oracle Database installation binaries are not stored in this GitHub repository.



Download Oracle Database 19c software separately from Oracle and place it in:



/u01/software



Example base software:



LINUX.X64\_193000\_db\_home.zip



Do not commit Oracle installation ZIP files or Oracle patch files to GitHub.



\---



\## Important Oracle Linux 9 Note



The original Oracle Database 19c base image is version 19.3.



Oracle Linux 9 requires a later Oracle Database 19c Release Update.



Do not treat the original 19.3 installation as the final supported Oracle home on Oracle Linux 9.



A supported Oracle 19c RU should be applied during or before completing the Oracle installation.



\---



\## GUI Installation with MobaXterm



Oracle Universal Installer can be displayed on Windows using X11 forwarding.



Connect to the VM using MobaXterm as the oracle user.



Check:



echo $DISPLAY



You should see something similar to:



localhost:10.0



Test X11:



xterm



If the graphical terminal appears on Windows, X11 forwarding is working.



The Oracle installer can then be started from:



cd $ORACLE\_HOME

./runInstaller



\---



\## Oracle Installation Directory



The Oracle Database 19c installation ZIP should be extracted directly into:



/u01/app/oracle/product/19.0.0/dbhome\_1



Example:



cd /u01/app/oracle/product/19.0.0/dbhome\_1

unzip /u01/software/LINUX.X64\_193000\_db\_home.zip



\---



\## Useful Vagrant Commands



Start the VM:



vagrant up



Check status:



vagrant status



Connect:



vagrant ssh



Gracefully stop:



vagrant halt



Suspend:



vagrant suspend



Resume:



vagrant resume



Destroy the VM:



vagrant destroy -f



Recreate the entire VM:



vagrant up



\---



\## Project Structure



Oracle-19c-vm/

├── Vagrantfile

├── README.md

└── .gitignore



Recommended .gitignore:



.vagrant/

\*.box

\*.zip

\*.log

software/



This prevents VM files, Oracle binaries, packaged Vagrant boxes, and installation files from being uploaded to GitHub.



\---



\## DBA Practice Goals



This VM can be used for Oracle DBA practice including:



\- Oracle Database 19c installation

\- Oracle Listener configuration

\- DBCA database creation

\- PDB and CDB administration

\- Users and roles

\- Tablespaces

\- Datafiles

\- Control files

\- Redo logs

\- Archive logging

\- FRA

\- RMAN backup and recovery

\- Data Pump

\- Flashback

\- Performance tuning

\- AWR / ASH

\- Oracle patching

\- OPatch

\- Oracle networking

\- Database startup and shutdown

\- Oracle services

\- Data Guard

\- Troubleshooting



\---



\## Purpose



This project provides a reproducible Oracle Database 19c lab environment.



Instead of manually creating and configuring an Oracle Linux VM every time, the operating system and Oracle prerequisites can be recreated using:



vagrant up



The VM can then be used for Oracle DBA installation, administration, backup, recovery, patching, performance tuning, and high-availability practice.



