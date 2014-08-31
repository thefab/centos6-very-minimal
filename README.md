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
	for SERVICE in auditd fcoe iptables ip6tables iscsi iscsid lldpad lvm2-monitor netfs nfslock rpcbind rpcidmapd rpcgssd; do
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

	# Clean cache
	yum clean all

	# Reboot
	reboot

## Conclusion

Very minimal list of processes (with an open ssh connection) :

	[root@centos6 ~]# ps -ef |wc -l
	63

	[root@centos6 ~]# pstree
	init─┬─dhclient
	     ├─login───bash
	     ├─6*[mingetty]
	     ├─rsyslogd───3*[{rsyslogd}]
	     ├─sshd───sshd───bash───pstree
	     └─udevd───2*[udevd]

Less than 50 MB of used memory :

	[root@centos6 ~]# free |grep buffers
                 total       used       free     shared    buffers     cached
	-/+ buffers/cache:      49684     971012

810 MB of used disk space :

	[root@centos6 ~]# df -h
	Filesystem            Size  Used Avail Use% Mounted on
	/dev/sda2              40G  809M   38G   3% /
	tmpfs                 499M     0  499M   0% /dev/shm

> A GOOD FOUNDATION TO BUILD ON !

# License (New BSD)

	Copyright (c) 2012, Fabien MARTY <fabien.marty@gmail.com>
	All rights reserved.

	Redistribution and use in source and binary forms, with or without
	modification, are permitted provided that the following conditions are met:
	    * Redistributions of source code must retain the above copyright
	      notice, this list of conditions and the following disclaimer.
	    * Redistributions in binary form must reproduce the above copyright
	      notice, this list of conditions and the following disclaimer in the
	      documentation and/or other materials provided with the distribution.
	    * Neither the name of the <organization> nor the
	      names of its contributors may be used to endorse or promote products
	      derived from this software without specific prior written permission.

	THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
	ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
	WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
	DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
	DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
	(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
	LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
	ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
	(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
	SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

