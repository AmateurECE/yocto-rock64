# Net-booting Raspberrypi4-64

After booting to U-boot:

```
U-boot>setenv serverip 192.168.1.60
U-boot>tftp ${kernel_addr_r} Image
U-boot>tftp ${fdt_addr_r} bcm2711-rpi-4-b.dtb
U-boot>booti ${kernel_addr_r} - ${fdt_addr_r}
```

# Using the eSDK container

Create a volume with the name of the SDK (I keep separate volumes for each SDK)

```
podman volume create cortexta72-poky-linux
```

Install the eSDK to the volume:

```
[edtwardy@Hackintosh ~]$ podman run -it --rm \
       -v cortexa72-poky-linux:/home/yocto/poky_sdk \
       -v $PWD:/home/yocto/installer \
       localhost/esdk-container
yocto@ea5526d4ee6e:~$ ./installer/poky-glibc-*.sh
```

Finally, add the SSTATE mirror, by manually editing
`~/poky_sdk/conf/local.conf` to include the following:

```
SSTATE_MIRRORS = "\
...
file://(.*) http://edtwardy-yocto.local/sstate-cache/\1 \
"
```

Also, make sure that there's an entry in `/etc/hosts` for
`edtwardy-yocto.local`. Or, the URL `https://edtwardy.hopto.org/sstate-cache`
can be used to allow remote access. 

After this, the container is configured to source the environment setup script
automatically, so all that's necessary is to spin up a container and develop!

```
podman run -it --rm -v cortexa72-poky-linux:/home/yocto/poky_sdk \
    localhost/esdk-container
```

One could even maintain different SDK's in different volumes and select the
desired SDK at container creation time.

In the past, I've had problems with UID ownership of files/directories while
using the container. To resolve this, I have to `chown` the necessary
filesystem items on the host to the UID/GID (which are the same) of the `yocto`
user in the container. To discover this user's UID/GID, try taking a look at
the SDK volume using `podman inspect cortexa72-poky-linux` and then running
`ls -l` on the directory where the volume resides on the host.

I find it's also useful to mount my `.gitconfig` inside the container. So, the
entire invocation looks like:

```
podman run -it --rm \
    -v cortexa72-poky-linux:/home/yocto/poky_sdk \
    -v $HOME/.gitconfig:/home/yocto/.gitconfig \
    -v $PWD:/home/yocto/app \
    localhost/esdk-container
```
