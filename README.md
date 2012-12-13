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

# Howto

## First stage

Install a standard CentOS x86_64 minimal distribution with (for example) : `CentOS-6.3-x86_64-minimal.iso`.

Then :

    # Install latest updates
	yum -y upgrade

	# Reboot
	reboot

## Second stage

FIXME

