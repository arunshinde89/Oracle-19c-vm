Vagrant.configure("2") do |config|

  # ============================================================
  # BASE OS
  # ============================================================

  config.vm.box = "generic/oracle9"

  config.vm.hostname = "oracle19c-vm"


  # ============================================================
  # NETWORK
  # ============================================================

  config.vm.network "private_network",
    ip: "192.168.136.20"


  # ============================================================
  # VMWARE
  # ============================================================

  config.vm.provider "vmware_desktop" do |v|

    # Open VMware console
    v.gui = true

    # RAM = 8 GB
    v.vmx["memsize"] = "8192"

    # CPU = 4
    v.vmx["numvcpus"] = "4"

  end


  # ============================================================
  # PROVISIONING
  # ============================================================

  config.vm.provision "shell", inline: <<-'SHELL'

    echo "============================================="
    echo " Oracle Linux 9 - Oracle 19c Lab Provisioning"
    echo "============================================="


    # ==========================================================
    # 1. UPDATE OPERATING SYSTEM
    # ==========================================================

    echo "Updating Oracle Linux..."

    dnf update -y


    # ==========================================================
    # 2. INSTALL BASIC UTILITIES
    # ==========================================================

    echo "Installing utilities..."

    dnf install -y \
      wget \
      unzip \
      tar \
      vim \
      nano \
      xfsprogs \
      net-tools \
      openssh-server


    # ==========================================================
    # 3. ROOT PASSWORD
    # ==========================================================

    echo "Setting root password..."

    echo 'root:welcome1' | chpasswd


    # ==========================================================
    # 4. SSH CONFIGURATION
    # ==========================================================

    echo "Configuring SSH..."

    sed -i \
      's/^[[:space:]]*PasswordAuthentication[[:space:]].*/# &/' \
      /etc/ssh/sshd_config

    sed -i \
      's/^[[:space:]]*PermitRootLogin[[:space:]].*/# &/' \
      /etc/ssh/sshd_config

    sed -i '1i PasswordAuthentication yes' /etc/ssh/sshd_config

    sed -i '1i PermitRootLogin yes' /etc/ssh/sshd_config


    if [ -d /etc/ssh/sshd_config.d ]; then

      grep -Rl '^PasswordAuthentication no' \
        /etc/ssh/sshd_config.d 2>/dev/null | \
        xargs -r sed -i \
        's/^PasswordAuthentication no/PasswordAuthentication yes/'

    fi


    sshd -t

    systemctl restart sshd


    # ==========================================================
    # 5. CREATE /u01 100 GB FILESYSTEM
    # ==========================================================

    echo
    echo "============================================="
    echo " Creating /u01 100 GB Filesystem "
    echo "============================================="


    mkdir -p /var/lib/oracledisks


    if [ ! -f /var/lib/oracledisks/u01.img ]; then

      echo "Creating 100 GB /u01 image..."

      truncate -s 100G /var/lib/oracledisks/u01.img

      mkfs.xfs -f /var/lib/oracledisks/u01.img

    fi


    mkdir -p /u01


    # ==========================================================
    # 6. CONFIGURE /u01 IN /etc/fstab
    # ==========================================================

    echo "Configuring /etc/fstab for /u01..."


    grep -q '/var/lib/oracledisks/u01.img' /etc/fstab || \
      echo '/var/lib/oracledisks/u01.img /u01 xfs loop,nofail 0 0' \
      >> /etc/fstab


    mount -a


    # ==========================================================
    # 7. CREATE 8 GB SWAP
    # ==========================================================

    echo
    echo "============================================="
    echo " Creating 8 GB Swap "
    echo "============================================="


    if [ ! -f /swapfile ]; then

      echo "Creating 8 GB swap file..."

      fallocate -l 8G /swapfile

      chmod 600 /swapfile

      mkswap /swapfile

    fi


    # Enable swap if it is not already active
    if ! swapon --show=NAME | grep -qx '/swapfile'; then

      swapon /swapfile

    fi


    # Make swap persistent after reboot
    grep -q '^/swapfile ' /etc/fstab || \
      echo '/swapfile swap swap defaults 0 0' \
      >> /etc/fstab


    echo
    echo "Swap configuration:"

    swapon --show

    free -h


    # ==========================================================
    # 8. INSTALL ORACLE DATABASE PREINSTALL PACKAGE
    # ==========================================================

    echo
    echo "============================================="
    echo " Installing Oracle 19c Preinstall Package "
    echo "============================================="


    dnf install -y oracle-database-preinstall-19c


    # ==========================================================
    # 9. SET ORACLE USER PASSWORD
    # ==========================================================

    echo "Setting oracle password..."

    echo 'oracle:welcome1' | chpasswd


    # ==========================================================
    # 10. CREATE ORACLE DIRECTORIES
    # ==========================================================

    echo
    echo "============================================="
    echo " Creating Oracle Directory Structure "
    echo "============================================="


    mkdir -p /u01/app/oracle/product/19.0.0/dbhome_1

    mkdir -p /u01/app/oraInventory

    mkdir -p /u01/software

    mkdir -p /u01/oradata

    mkdir -p /u01/fast_recovery_area


    # ==========================================================
    # 11. ORACLE OWNERSHIP
    # ==========================================================

    chown -R oracle:oinstall /u01

    chmod -R 775 /u01


    # ==========================================================
    # 12. ORACLE ENVIRONMENT
    # ==========================================================

    echo
    echo "============================================="
    echo " Configuring Oracle Environment "
    echo "============================================="


    if ! grep -q 'ORACLE_BASE=/u01/app/oracle' \
      /home/oracle/.bash_profile; then

      cat >> /home/oracle/.bash_profile <<'EOF'

# ============================================================
# Oracle Database 19c Environment
# ============================================================

export ORACLE_BASE=/u01/app/oracle

export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1

export ORACLE_SID=ORCL

export PATH=$ORACLE_HOME/bin:$PATH

export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib

export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib

EOF

    fi


    chown oracle:oinstall /home/oracle/.bash_profile


    # ==========================================================
    # 13. VERIFY ORACLE USER
    # ==========================================================

    echo
    echo "============================================="
    echo " Oracle User "
    echo "============================================="


    id oracle


    # ==========================================================
    # 14. VERIFY ORACLE GROUPS
    # ==========================================================

    echo
    echo "============================================="
    echo " Oracle Groups "
    echo "============================================="


    groups oracle


    # ==========================================================
    # 15. VERIFY /u01
    # ==========================================================

    echo
    echo "============================================="
    echo " /u01 Filesystem "
    echo "============================================="


    df -hT /u01


    # ==========================================================
    # 16. VERIFY SWAP
    # ==========================================================

    echo
    echo "============================================="
    echo " Swap Configuration "
    echo "============================================="


    swapon --show

    free -h


    # ==========================================================
    # 17. VERIFY KERNEL PARAMETERS
    # ==========================================================

    echo
    echo "============================================="
    echo " Oracle Kernel Parameters "
    echo "============================================="


    sysctl fs.aio-max-nr

    sysctl fs.file-max

    sysctl kernel.shmmax

    sysctl kernel.sem


    # ==========================================================
    # 18. VERIFY ORACLE LIMITS
    # ==========================================================

    echo
    echo "============================================="
    echo " Oracle User Limits "
    echo "============================================="


    grep oracle /etc/security/limits.d/* 2>/dev/null || true


    # ==========================================================
    # FINAL STATUS
    # ==========================================================

    echo
    echo "============================================="
    echo " Oracle Linux 9 Stage-1 VM Ready "
    echo "============================================="

    echo
    echo "Hostname       : oracle19c-vm"
    echo "Private IP     : 192.168.136.20"
    echo
    echo "Operating Sys  : Oracle Linux 9"
    echo
    echo "RAM            : 8 GB"
    echo "CPU            : 4"
    echo "Swap           : 8 GB"
    echo
    echo "/u01           : 100 GB"
    echo
    echo "Oracle User    : oracle"
    echo "Oracle Group   : oinstall"
    echo
    echo "ORACLE_BASE    : /u01/app/oracle"
    echo "ORACLE_HOME    : /u01/app/oracle/product/19.0.0/dbhome_1"
    echo "ORACLE_SID     : ORCL"
    echo
    echo "Software Stage : /u01/software"
    echo "Data Location  : /u01/oradata"
    echo "FRA Location   : /u01/fast_recovery_area"
    echo
    echo "============================================="

  SHELL

end