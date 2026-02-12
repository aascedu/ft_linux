I made this file to keep track of the cmds or ways to create the host that will have Vagrant installed.

Create the QEMU image : `qemu-img create -f qcow2 ~/goinfre/iot-vm.qcow2 40G`

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


Clean sources dir one liner:
```bash
for FILE in $(ls); do
  file $FILE | grep 'directory' > /dev/null
  if [[ $? == 0 ]]; then
    echo $FILE
    rm -r $FILE
  fi
done
```


compress vm :
```bash
#!/bin/bash
set -e

MOUNTPOINTS=(
    "/mnt/lfs/boot"
    "/mnt/lfs"
    "/"
)


if [[ $EUID -ne 0 ]]; then
    exit 1
fi

# SWAP_DEVICE=$(swapon --show=NAME --noheadings)

# if [[ -n "$SWAP_DEVICE" ]]; then
#     echo -e "\nProcessing $SWAP_DEVICE..."
#     swapoff "$SWAP_DEVICE"

#     dd if=/dev/zero of="$SWAP_DEVICE" bs=1M status=progress 2>/dev/null || true

#     mkswap -L LFS_SWAP "$SWAP_DEVICE"

#     swapon "$SWAP_DEVICE"
#     echo -e "$SWAP_DEVICE processed"
# else
#     echo -e "No active swap detected\n"
# fi

LFS=/mnt/lfs

sync
echo 3 > /proc/sys/vm/drop_caches

find /var/log -type f -name "*.gz" -delete 2>/dev/null || true
find $LFS/var/log -type f -name "*.[0-9]" -delete 2>/dev/null || true

rm -rf /tmp/* /var/tmp/* 2>/dev/null || true
rm -rf $LFS/tmp/* $LFS/var/tmp/* 2>/dev/null || true

for mp in "${MOUNTPOINTS[@]}"; do
    if mountpoint -q "$mp" 2>/dev/null; then
        echo -e "\nProcessing $mp..."

        AVAIL=$(df -h "/dev/sda1" | awk '{print $4}' | tail -1)
        echo "Available space: $AVAIL"

        dd if=/dev/zero of="${mp}/zerofill" bs=1M status=progress 2>&1 || true
        sync
        rm -f "${mp}/zerofill"
        sync

        echo -e "$mp processed"
    else
        echo -e "$mp is not a mount point, ignored"
    fi
done

sync

echo -e "\nZero-fill complete"
echo -e "qemu-img convert -O qcow2 -c disk.qcow2 disk-compressed.qcow2"
```