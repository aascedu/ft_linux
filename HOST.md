I made this file to keep track of the cmds or ways to create the host that will have Vagrant installed.

Create the QEMU image : qemu-img create -f qcow2 ~/goinfre/iot-vm.qcow2 60G

Then the host with QEMU :
```bash
  qemu-system-x86_64 \
  -enable-kvm \
  -m 8G \
  -smp 20 \
  -cpu max \
  -hda ~/goinfre/iot-vm.qcow2 \
  -cdrom ~/Downloads/ubuntu-24.04-server.iso \
  -boot d \
  -vga virtio \
  -nic user,hostfwd=tcp::2222-:22
```

Then to launch the VM :

```bash
qemu-system-x86_64 \
  -enable-kvm \
  -m 8192 \
  -smp 20 \
  -cpu max \
  -hda ~/goinfre/iot-vm.qcow2 \
  -nic user,hostfwd=tcp::2222-:22 \
  -vga virtio
```


To validate version-check.sh :

```
sudo ln -sf /bin/bash /bin/sh
sudo apt install -y build-essential texinfo make gcc binutils bison
```

Oneliner to download packages needed :
```bash
cd $LFS/sources
wget -r -np -nH --cut-dirs=4 -R "index.html*" https://mirror.koddos.net/lfs/lfs-packages/12.4/
```