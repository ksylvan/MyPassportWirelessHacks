# Download the binary

If you just want to grab the binary, it's in the [binary/](https://github.com/ksylvan/MyPassportWirelessHacks/tree/master/ncat/binary) subdirectory
here.  Do so at your own risk. It works for me, but I make no
guarantees that it won't harm your device.

The rest of this guide walks you through creating your own
cross-compiling toolchain and compiling netcat from source for your My
Passport Wireless device.

# Cross-compilng ncat for the My Passport Wireless

These are the exact steps I used to cross-compile netcat from
http://nmap.org/ncat/ for the My Passport Wireless network attached
storage device. My host system was a Fedora 20 Linux, but similar
steps will work for other distributions.

## Setup crosstool-ng:

    git clone git://crosstool-ng.org/crosstool-ng
    ./bootstrap
    ./configure --enable-local

The configure step fails if you are missing pre-requisites. So at this point,
install whatever is missing. For my Fedora 20 system, I had to do:

    sudo yum install gperf
    sudo yum install flex

Then go back to the rest of the steps:

    ./configure --enable-local 
    ./ct-ng menuconfig

At this point, you have to match the crosstool-ng toolchain with your
My Passport Wireless device. I had gathered some information from my system:

    # cat /etc/os-release 
    NAME=Buildroot
    VERSION=2013.05-svn157-dirty
    ID=buildroot
    VERSION_ID=2013.05
    PRETTY_NAME="Buildroot 2013.05"
    
    # cat /proc/version 
    Linux version 3.2.0 (primax@primax-vm) (gcc version 4.7.3 20130226 (prerelease) (crosstool-NG linaro-1.13.1-4.7-2013.03-20130313 - Linaro GCC 2013.03) ) #1 Thu Oct 2 16:08:39 CST 2014
    
    # uname -a
    Linux Home2 3.2.0 #1 Thu Oct 2 16:08:39 CST 2014 armv7l GNU/Linux
    
    # file /bin/busybox 
    /bin/busybox: setuid ELF 32-bit LSB  executable, ARM, EABI5 version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.31, BuildID[sha1]=7508def24d66e7eaffa9853bd9e099fe0aa93d00, stripped

Based on that information, I set up my toolchain in the menuconfig as follows:

* Target options ---> Target Architecture ---> select arm
* Operating System ---> Target OS ---> select linux
* Linux kernel version ---> select 2.6.31.14
* C-library ---> C library ---> select eglibc
* C-library ---> eglibc version ---> select 2_15

After you do all the above, you can exit all the way out of menuconfig,
which will write your .config file.

I have included my .config file (as support/config) in this repository,
in case you have problems, but I make no guarantees that it will work
for you.

Then, continuing with the build:

    ./ct-ng build

if you get a message about LD_LIBRARY_PATH, then

    unset LD_LIBRARY_PATH
    ./ct-ng build

This step will take a while. When it's finished, you will have a
directory ~/x-tools that contains your cross-compiling toolchain.

    $ ls ~/x-tools/
    arm-unknown-linux-gnueabi
    
    $ ls ~/x-tools/arm-unknown-linux-gnueabi
    
    $ ls ~/x-tools/arm-unknown-linux-gnueabi
    arm-unknown-linux-gnueabi  bin  build.log.bz2  include  lib  libexec  share
    
    $ ls ~/x-tools/arm-unknown-linux-gnueabi/bin/
    arm-unknown-linux-gnueabi-addr2line     arm-unknown-linux-gnueabi-gprof
    arm-unknown-linux-gnueabi-ar            arm-unknown-linux-gnueabi-ld
    arm-unknown-linux-gnueabi-as            arm-unknown-linux-gnueabi-ld.bfd
    arm-unknown-linux-gnueabi-cc            arm-unknown-linux-gnueabi-ldd
    arm-unknown-linux-gnueabi-c++filt       arm-unknown-linux-gnueabi-nm
    arm-unknown-linux-gnueabi-cpp           arm-unknown-linux-gnueabi-objcopy
    arm-unknown-linux-gnueabi-ct-ng.config  arm-unknown-linux-gnueabi-objdump
    arm-unknown-linux-gnueabi-elfedit       arm-unknown-linux-gnueabi-populate
    arm-unknown-linux-gnueabi-gcc           arm-unknown-linux-gnueabi-ranlib
    arm-unknown-linux-gnueabi-gcc-4.7.3     arm-unknown-linux-gnueabi-readelf
    arm-unknown-linux-gnueabi-gcc-ar        arm-unknown-linux-gnueabi-size
    arm-unknown-linux-gnueabi-gcc-nm        arm-unknown-linux-gnueabi-strings
    arm-unknown-linux-gnueabi-gcc-ranlib    arm-unknown-linux-gnueabi-strip
    arm-unknown-linux-gnueabi-gcov

As you can see, crosstool-ng creates some unwieldy file names.  So, I
wrote a shell script (included as support/x-tools-symlink). This
script creates a bin directory in ~/x-tools populated by symlinks that
point to the cross-compiling toolchain binaries:

    $ ./support/x-tools-symlink arm-unknown-linux-gnueabi
    Setting up bin directory.
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-addr2line addr2line
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-ar ar
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-as as
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-cc cc
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-c++filt c++filt
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-cpp cpp
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-ct-ng.config ct-ng.config
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-elfedit elfedit
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-gcc gcc
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-gcc-4.7.3 gcc-4.7.3
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-gcc-ar gcc-ar
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-gcc-nm gcc-nm
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-gcc-ranlib gcc-ranlib
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-gcov gcov
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-gprof gprof
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-ld ld
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-ld.bfd ld.bfd
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-ldd ldd
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-nm nm
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-objcopy objcopy
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-objdump objdump
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-populate populate
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-ranlib ranlib
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-readelf readelf
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-size size
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-strings strings
    ln -s ../arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-strip strip
    Done.

Now you can simply set your PATH to point to ~/x-tools/bin first,
which will use your cross-compiling toolchain binaries with the usual
names (like "gcc" and "ld"), obviating the need to modify configure scripts and Makefiles.

## Compiling ncat

With our toolchain ready to use, we are now ready to compile ncat.

Grab the nmap sources:

    svn co https://svn.nmap.org/nmap
    cd nmap
    PATH=~/x-tools/bin:$PATH
    ./configure --host=arm-linux --with-pcap=null
    cd libpcap
    make
    cd ../ncat/
    make

Now you have a binary that will run on your My Passport Wireless!

    $ file ncat
    ncat: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.31, not stripped
    $ ls -l ncat
    -rwxrwxr-x. 1 ksylvan ksylvan 1189971 Oct 20 13:48 ncat

Let's make it smaller:

    $ strip ncat
    $ file ncat
    ncat: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.31, stripped
    $ ls -l ncat
    -rwxrwxr-x. 1 ksylvan ksylvan 288988 Oct 20 13:49 ncat

Now you can copy this binary to your device and run it:

    # uname -a
    Linux Home2 3.2.0 #1 Thu Oct 2 16:08:39 CST 2014 armv7l GNU/Linux
    # ./ncat --version
    Ncat: Version 6.47SVN ( http://nmap.org/ncat )

# Tricks with ncat

I have a UVerse gateway at my home for internet connectivity. The
administrative interface is pretty limited, but it allows me to
port-forward specific ports to other devices in the network via its
Firewall->"NAT/Gaming" tab. Using this, I forwarded port 22 (ssh) to
my My Passport Wireless device.

Armed with the external WAN address of my uVerse gateway, represented
as AA.BB.CC.DD below, I can now do this sort of thing:

From my laptop:

     $ ncat --sh-exec "ssh root@AA.BB.CC.DD ./ncat 192.168.1.254 80" --keep-open -l 8080

This uses the ncat I copied over ("./ncat") to connect to
192.168.1.254:80 which is the local area network address of the web
interface for the uVerse gateway and makes it available on my laptop at localhost:8080.

Then I use a browser and connect to localhost:8080 and I can
administer the uVerse gateway.

In fact, using similar commands, I can access all sorts of internal resources:

Suppose I have a Linux box in my internal network at 192.168.2.2.

From my laptop, from an external address, I can do this:

    $ ncat --sh-exec "ssh root@AA.BB.CC.DD ./ncat 192.168.2.2 22" --keep-open -l 1122

And then, I can use the -p option of ssh to login to that machine:

    $ ssh -o StrictHostKeyChecking=false -p 1122 root@localhost
    Warning: Permanently added '[localhost]:1122' (ECDSA) to the list of known hosts.
    #
