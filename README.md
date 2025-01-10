# luckfox-pico-alpine
buildroot wasn't too convenient, lot of packages weren't working

ubuntu used too much ram so i couln't reserve enough ram for camera

so i decided to use alpine linux for my project


how to create rootfs img

this should create very minimal rootfs and will not be able to host ssh out of box

so you have to do setup on serial tty

prepare apk-tools, (native or static or whatever)

i don't recommend doing on foreign arch tho

huge thanks to @ptrcnull from alpine linux telegram
## creating image 
### create image file
```
fallocate -l 200M rootfs.img
mkfs.ext4 rootfs.img
```
### mount image
```
mkdir alpine
mount rootfs.img alpine
```

### extract alpine rootfs
```
cd alpine
curl https://dl-cdn.alpinelinux.org/alpine/v3.21/releases/armhf/alpine-minirootfs-3.21.2-armhf.tar.gz | tar -xzv
cd ..
```

### install required packages
```
apk --root alpine add $(cat packages)
```

### apply overlay
```
mkdir overlay
cd overlay
tar -xzvf ../overlay.tar.gz
cd ..
rsync -a overlay/ alpine/
```


### unmount 
```
umount alpine
```

now you can flash rootfs.img

## flashing
use images from buildroot except rootfs.img

## setup
### remount root
you have to remount root rw only on first boot
```
mount -o remount,rw /
```

### setup-alpine
```
setup-alpine
reboot
```
this will setup basic services including sysfs etc

### resize root
```
resize2fs /dev/root
```

### setup-devd
```
setup-devd
```

### install drivers as you want
if you want to install driver using usb storage, you can use luckfox-config

### do setup-alpine again after installing driver
but make sure you enabled swap before doing this
```
this will allow you to setup network
```


## using camera
alpine linux uses musl libc, camera can only work on uClibc environment

however i included uClibc environment to the image(`/uclibc`)

you have to patch the binary that's built for uClibc and it will run on this image

```
patchelf --set-interpreter /uclibc/ld-uClibc.so.0 --set-rpath /uclibc/ <your executable file>
```
