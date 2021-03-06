# NVMe Emulator for SPDK in Docker

[![Build Status](https://travis-ci.org/ljishen/nvme-env.svg?branch=master)](https://travis-ci.org/ljishen/nvme-env)
[![](https://images.microbadger.com/badges/image/ljishen/spdk-vagrant.svg)](http://microbadger.com/images/ljishen/spdk-vagrant)

This work packages this [Vagrant Development Environment](http://www.spdk.io/doc/vagrant.html) configured for project Storage Performance Development Kit (SPDK) into a docker container.


## Prerequisite

Check if your Intel CPU support the [SSSE3](https://en.wikipedia.org/wiki/SSSE3) instruction set
```bash
grep '\bssse3\b' /proc/cpuinfo
```


## Usage

1. Install the kernel headers on Ubuntu or Debian Linux
   ```bash
   sudo apt-get install linux-headers-$(uname -r)
   ```

1. Build the VirtualBox kernel modules **only once** before the first time your launch the VM
   ```bash
   docker run -ti \
       --privileged \
       -v /usr/src:/usr/src \
       -v /lib/modules:/lib/modules \
       ljishen/spdk-vagrant \
       /sbin/vboxconfig
   ```
   This process shows the error message **Failed to connect to bus: No such file or directory** for several times which are expected and harmless. These errors come from calling `systemctl daemon-reexec` in the startup script. According to the [man page](http://man7.org/linux/man-pages/man1/systemctl.1.html) for `systemctl`, the command `daemon-reexec` "is of little use except for debugging and package upgrades".

1. Launch the VM
   ```bash
   docker run -ti \
       --privileged \
       --net host \
       ljishen/spdk-vagrant
   ```
   This gonna take a while since it sets up the network, installs all the dependencies and also builds the SPDK.

1. Now you can try the NVMe sample application "Hello World" as mentioned in the [doc](https://github.com/spdk/spdk/blob/master/scripts/vagrant/README.md#hello-world)
   ```bash
   Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-21-generic x86_64)

    * Documentation:  https://help.ubuntu.com/
   vagrant@localhost:~$ lspci | grep "Non-Volatile"
   00:0e.0 Non-Volatile memory controller: InnoTek Systemberatung GmbH Device 4e56

   vagrant@localhost:~$ sudo /spdk/examples/nvme/hello_world/hello_world
   Starting DPDK 17.08.0 initialization...
   [ DPDK EAL parameters: hello_world -c 0x1 --file-prefix=spdk0 --base-virtaddr=0x1000000000 --proc-type=auto ]
   EAL: Detected 4 lcore(s)
   EAL: Auto-detected process type: PRIMARY
   EAL: Probing VFIO support...
   Initializing NVMe Controllers
   EAL: PCI device 0000:00:0e.0 on NUMA socket 0
   EAL:   probe driver: 80ee:4e56 spdk_nvme
   Attaching to 0000:00:0e.0
   Attached to 0000:00:0e.0
   Using controller ORCL-VBOX-NVME-VER12 (VB1234-56789        ) with 1 namespaces.
     Namespace ID: 1 size: 1GB
   Initialization complete.
   Hello world!
   ```

1. You can `exit` the VM and learn how to control the VM by typing `vagrant --help`, e.g.
   ```bash
   # vagrant status
   Current machine states:

   default                   running (virtualbox)

   The VM is running. To stop this VM, you can run `vagrant halt` to
   shut it down forcefully, or you can run `vagrant suspend` to simply
   suspend the virtual machine. In either case, to restart it again,
   simply run `vagrant up`.
   ```


## VM Configuration

By default, the VM boots with:

- Ubuntu 16.04 LTS (GNU/Linux 4.4.0-21-generic x86_64)
- 4GM memory
- 4 virtual CPU

The following instructions help to change the default resource configuration if you want:

1. Launch the **bash** shell
   ```bash
    docker run -ti \
        --privileged \
        --net host \
        -v /usr/src:/usr/src \
        -v /lib/modules:/lib/modules \
        ljishen/spdk-vagrant \
        shell
   ```

1. Edit the file `/root/spdk/scripts/vagrant/env.sh` to change the value of `SPDK_VAGRANT_VMCPU` or `SPDK_VAGRANT_VMRAM`, then save and exit.

1. Run `/sbin/vboxconfig` if your didn't launch the VM before.

1. Launch the VM with `up`.


## Troubleshooting

* **VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005)**

  You host system may have an out-of-data VirtualBox installed. Please either upgrade or uninstall it.


## Tested Environment

* Docker Version >= 1.12.5
* Ubuntu Release >= 12.04
