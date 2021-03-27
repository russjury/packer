# Create VM image (QCOW2 for QEMU/KVM) using Packer
The examples here will create a qcow2 image for use with QEMU/KVM virtualization using [Hashicorp's Packer](https://www.packer.io/).  They start with a CentOS7 or CentOS8 ISO and a kickstart file.  All that's needed for Packer is the QEMU libs/binaries -- it runs everything itself, so you don't have to actually be running libvirtd (handy for a build server or other CI/CD), and runs headless, meaning everything here can be run from command-line or scripts/automation.

To build an image, just:
```
# ./packer build centos7-minimal.json
```
or... 
```
# ./packer build centos8-minimal.json
```

Pieces missing from this repo are the [CentOS7(minimal) or CentOS8(dvd1) ISO](https://www.centos.org/download/), and the [packer binary](https://www.packer.io/downloads).  Note that Packer starts up its own webserver even, so all you have to do is put the kickstart file where Packer can find it (/e/packer/http in my case).

You can deploy a VM using the image packer just build like this (for QEMU/KVM):
```
# HOSTNAME=myhost1
# cp /e/packer/centos7/packer-qemu /e/VMs/$HOSTNAME.qcow2
# virt-install --vcpus=2 --memory=8192 --cpu host --machine q35 --os-variant centos7.0 --network bridge=br0 --graphics none --noautoconsole --sound none --console pty,target.type=virtio --serial pty --import --disk /e/VMs/$HOSTNAME.qcow2 --name $HOSTNAME
```
or...
```
# HOSTNAME=myhost1
# cp /e/packer/centos8/packer-qemu /e/VMs/$HOSTNAME.qcow2
# virt-install --vcpus=2 --memory=8192 --cpu host --machine q35 --os-variant centos8 --network bridge=br0 --graphics none --noautoconsole --sound none --console pty,target.type=virtio --serial pty --import --disk /e/VMs/$HOSTNAME.qcow2 --name $HOSTNAME
```
**NOTE:** This does not set the hostname inside the VM, and it relies on DHCP -- general okay if you're using Ansible, etc to configure the VM from this point on.

Also notice that the Packer JSON config doesn't rely on an SSH user that you have to remember to remove.  Kickstart is used to configure the VM instead of Packer SSH communicators.
