---
title: KVM基础
tags: |-

  - 云计算
  - Linux操作运维
permalink: kvmji-chu
id: 13
updated: '2016-09-22 15:04:50'
date: 2016-09-19 17:50:47
---

##KVM基础
###KVM虚拟机安装

	yum install qemu-kvm	
	yum install qemu-img.x86_64
	libvirt libvirt-devel  libvirt-client.x86_64

	安装以上软件需要OS支持

	qemu-kvm是虚拟化KVM套件
	libvirt是全套的虚拟化软件，并且可以使用virsh管理虚拟机
	
	定义一个虚拟机：
		virsh define appserver.xml
	启动一个虚拟机：
		virsh start appserver
	进入console:
		virsh console appserver

	直接通过命令启动虚拟机：
	先生成
	qemu-img create -f qcow2 test02.img 7G
	
	virt-install --name=oeltest02 --os-variant=RHEL5.8 --ram 512 --vcpus=1 --disk path=/data/test02.img,format=qcow2,size=7,bus=virtio --accelerate --cdrom /data/iso/oel58x64.iso --vnc --vncport=5910 --vnclisten=0.0.0.0 --network bridge=br0,model=virtio --noautoconsole

	virt-install --name=centos1 --ram 512 --vcpus=1 --disk path=/etc/libvirt/centos1.qcow2,format=qcow2,size=20,bus=virtio --cdrom /iso/CentOS-6.4-i386-LiveCD.iso --vnc --vncport=5910 --vnclisten=0.0.0.0 --network bridge=br0,model=virtio --noautoconsole


SR-IOV
http://www.cnblogs.com/sammyliu/p/4548194.html
