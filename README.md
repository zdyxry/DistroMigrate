# DistroMigrate

## Manual: CentOS 7 upgrade to CentOS8

```
# Update to CentOS 7 latest version
yum -y update
reboot

# Install rpm tools
yum -y install epel-release
yum -y install yum-utils 
yum -y install rpmconf
rpmconf -a

# Clean useless rpm
package-cleanup --leaves
package-cleanup --orphans

# Install dnf and uninstall yum
yum -y install dnf
dnf -y remove yum yum-metadata-parser
rm -Rf /etc/yum

# Install CentOS 8 release files
dnf -y upgrade
dnf -y install http://vault.centos.org/8.5.2111/BaseOS/x86_64/os/Packages/{centos-linux-repos-8-3.el8.noarch.rpm,centos-linux-release-8.5-1.2111.el8.noarch.rpm,centos-gpg-keys-8-3.el8.noarch.rpm}
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

# Upgrade epel release files
dnf -y upgrade https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf clean all

# Uninstall kernel
rpm -e `rpm -q kernel`
rpm -e --nodeps sysvinit-tools

# DNF distro-sync
dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync
# Key imported successfully
# Running transaction check
# Error: transaction check vs depsolve:
# (NetworkManager >= 1.20 or dhclient) is needed by dracut-network-049-191.git20210920.el8.x86_64
# rpmlib(RichDependencies) <= 4.12.0-1 is needed by dracut-network-049-191.git20210920.el8.x86_64
# To diagnose the problem, try running: 'rpm -Va --nofiles --nodigest'.
# You probably have corrupted RPMDB, running 'rpm --rebuilddb' might fix the issue.
# The downloaded packages were saved in cache until the next successful transaction.
# You can remove cached packages by executing 'dnf clean packages'.

# Resolve RPM conflicts
curl https://vault.centos.org/8.5.2111/BaseOS/x86_64/os/Packages/dracut-network-049-191.git20210920.el8.x86_64.rpm -o dracut-network-049-191.git20210920.el8.x86_64.rpm
rpm -Uvh dracut-network-049-191.git20210920.el8.x86_64.rpm --force --nodeps
rpm -e python36-rpmconf-1.0.22-1.el7.noarch --nodeps

# DNF distro-sync
dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync

# Install kernel and minimal package group
dnf -y install kernel-core
dnf -y groupupdate "Core" "Minimal Install"

# Reboot
reboot
```
