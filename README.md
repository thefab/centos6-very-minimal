# centos6-very-minimal

## What is it ?

A script / documentation to get a **very minimal** CentOS 6 x86_64 install to run in `VirtualBox`.

It's a just a base to build on.

## Rules

Here are some rules :

- no `IPV6`
- no `SELinux`, no firewall
- no specific hardware drivers (we run it in `VirtualBox`)
- we keep `yum` and a "not broken" rpm database 
- just ssh access
- no "VirtualBox guest additions"
- no `NFS`
- no `RPC`

# Howto

## First stage

Install a standard CentOS x86_64 minimal distribution with (for example) : `CentOS-6.3-x86_64-minimal.iso`.

Then :

    # Install latest updates
	yum -y upgrade

	# Clean cache
	yum clean all

	# Reboot
	reboot

Starting point :

- 214 packages
- around 80 MB of used memory
- 870 MB of used disk space

## Second stage

	# No SELinux
	cp -pf /etc/sysconfig/selinux /etc/sysconfig/selinux.orig
	cat /etc/sysconfig/selinux.orig |sed 's/^SELINUX=.*$/SELINUX=disabled/g' >/etc/sysconfig/selinux

	# No IPV6
	# (some ideas stolen on http://wiki.centos.org/FAQ/CentOS6)
	cp -pf /etc/sysctl.conf /etc/sysctl.conf.orig
	echo "net.ipv6.conf.all.disable_ipv6 = 1" >>/etc/sysctl.conf
	echo "net.ipv6.conf.default.disable_ipv6 = 1" >>/etc/sysctl.conf
	cp -pf /etc/hosts /etc/hosts.orig
	cat /etc/hosts.orig |grep -v localhost6 >/etc/hosts
	cp -pf /etc/ssh/sshd_config /etc/ssh/sshd_config.orig
	cat /etc/ssh/sshd_config |sed 's/#AddressFamily any/AddressFamily inet/g' |sed 's/#ListenAddress 0.0.0.0/ListenAddress 0.0.0.0/g' >/etc/ssh/sshd_config

	# Disable some services
	for SERVICE in auditd fcoe iptables ip6tables iptables iscsi iscsid lldpad lvm2-monitor netfs nfslock rpcbind rpcidmapd rpcgssd; do
	    chkconfig ${SERVICE} off
	done

	# Remove some packages
	cat >/tmp/packages_to_remove <<EOF
    aic94xx-firmware
	bfa-firmware
	kernel-firmware
	ql2100-firmware
	ql2200-firmware
	ql23xx-firmware
	ql2400-firmware
	ql2500-firmware
	iptables-ipv6
	nfs-utils
	nfs-utils-lib
	iscsi-initiator-utils
	device-mapper-multipath-libs
	device-mapper-multipath
	audit
	checkpolicy
	selinux-policy
	selinux-policy-targeted
	policycoreutils
	device-mapper
	cryptsetup-luks
	cryptsetup-luks-libs
	device-mapper-event
	device-mapper-event-libs
	device-mapper-libs
	kpartx
	lvm2
	lvm2-libs
	EOF
	yum remove `cat /tmp/packages_to_remove |xargs`

	[root@centos6 ~]# pstree
init─┬─dhclient
     ├─login───bash
     ├─5*[mingetty]
     ├─rsyslogd───3*[{rsyslogd}]
     ├─sshd───sshd───bash───pstree
     └─udevd───2*[udevd]





