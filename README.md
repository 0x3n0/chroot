# Chroot liveCD or other linux distro

The thing about chroots and `/proc`, `/sys` and `/dev/pts` is that these three filesystems are provided by the kernel, so they remain the same whether you mount within the chroot or from without. Indeed, you'll see, earlier on in the instructions:

```bash
sudo mount --bind /dev chroot/dev
```

`/dev` is populated by the kernel, but is not a kernel-provided filesystem, so it had to be bind-mounted. Therefore, in practice, you'll see that mounting it using bind mounts (or otherwise) before entering the chroot works just as well (assume `sudo`):

```bash
for i in dev proc sys dev/pts
do
    mount -o bind /$i chroot/$i
done
chroot chroot
for i in dev/pts proc sys dev
do
    umount -chroot/$i
done
# or
mount -o bind /dev chroot/dev
mount -t sysfs none chroot/sys
mount -t proc none chroot/proc
mount -t devpts none chroot/dev/pts
chroot chroot
for i in dev/pts proc sys dev
do
    umount -chroot/$i
done
```

You could create a .bashrc script or something like it, which is appended to the chroot env's /root/.bashrc, which does all the mounting etc. Aftwerwards you restore the backed up .bashrc in /root and exit the chroot:

```bash
#!/usr/bin/env bash
cp bashrcscript chroot/root/
if [ -a chroot/root/.bashrc ]; then
    cp chroot/root/.bashrc chroot/root/.bashrc.bak
fi
echo "./bashrcscript" >> chroot/root/.bashrc
chroot chroot/
rm chroot/root/.bashrc
rm chroot/root/bashrcscript
if [ -a chroot/root/.bashrc.bak ]; then
    mv chroot/root/.bashrc.bak chroot/root/.bashrc
fi
```

bashrc script for mount:

```plaintext
mount none -t proc /proc
mount none -t sysfs /sys
mount none -t devpts /dev/pts 
```

The bashrcscript will then be executed when the root console is started. Ensure it's executable.
