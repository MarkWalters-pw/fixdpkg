# fixdpkg
Extracts a debian package, runs your custom script on it, then repackages it.


patch-libpng12 is an example of what you can do with fixdpkg.  It will allow you to install libpng12 on newer versions of Ubuntu.

This is the error you get when trying to install libpng12 on Ubuntu focal.
```
$ apt -y install libpng12-0
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Package libpng12-0 is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source

E: Package 'libpng12-0' has no installation candidate
$ wget -cq http://mirrors.kernel.org/ubuntu/pool/main/libp/libpng/libpng12-0_1.2.54-1ubuntu1_amd64.deb
$ dpkg -i libpng12-0_1.2.54-1ubuntu1_amd64.deb 
Selecting previously unselected package libpng12-0:amd64.
(Reading database ... 346483 files and directories currently installed.)
Preparing to unpack libpng12-0_1.2.54-1ubuntu1_amd64.deb ...
Unpacking libpng12-0:amd64 (1.2.54-1ubuntu1) ...
dpkg: error processing archive libpng12-0_1.2.54-1ubuntu1_amd64.deb (--install):
 unable to install new version of '/lib/x86_64-linux-gnu/libpng12.so.0': No such file or directory
Processing triggers for libc-bin (2.30-0ubuntu3) ...
Errors were encountered while processing:
 libpng12-0_1.2.54-1ubuntu1_amd64.deb
```

To fix this you must move the library to the correct directory and add a symbolic link.

```
cat <<EOF | ./fixdpkg -i -p http://mirrors.kernel.org/ubuntu/pool/main/libp/libpng/libpng12-0_1.2.54-1ubuntu1_amd64.deb
rm -rf usr/share usr/lib/x86_64-linux-gnu/*
mv lib/x86_64-linux-gnu/libpng12.so.0.54.0 usr/lib/x86_64-linux-gnu
rm -rf lib
cd usr/lib/x86_64-linux-gnu
ln -s libpng12.so.0.54.0 libpng12.so.0
EOF
```

Which results in:
```
Downloading libpng12-0_1.2.54-1ubuntu1_amd64.deb
Extracting package
Running patch
Repackaging 
dpkg-deb: building package 'libpng12-0' in 'libpng12-0_1.2.54-1ubuntu1_amd64.deb-patched.deb'.
(Reading database ... 346483 files and directories currently installed.)
Preparing to unpack libpng12-0_1.2.54-1ubuntu1_amd64.deb-patched.deb ...
Unpacking libpng12-0:amd64 (1.2.54-1ubuntu1) ...
Setting up libpng12-0:amd64 (1.2.54-1ubuntu1) ...
Processing triggers for libc-bin (2.30-0ubuntu3) ...
```

Another way to use it is to call its functions from your script.

```
. fixdpkg

url="http://mirrors.kernel.org/ubuntu/pool/main/libp/libpng/libpng12-0_1.2.54-1ubuntu1_amd64.deb"
pkg="${url##*/}"
wget -cq "${url}"

extractpkg "${pkg}"

rm -rf usr/share usr/lib/x86_64-linux-gnu/*
mv lib/x86_64-linux-gnu/libpng12.so.0.54.0 usr/lib/x86_64-linux-gnu
rm -rf lib
cd usr/lib/x86_64-linux-gnu || throw "Invalid directory"
ln -s libpng12.so.0.54.0 libpng12.so.0

repackage

sudo dpkg -i ${pkg%%.deb}-patched.deb
```

Which reults in:

```
$ patch-libpng12-v2
dpkg-deb: building package 'libpng12-0' in 'libpng12-0_1.2.54-1ubuntu1_amd64-patched.deb'.
Selecting previously unselected package libpng12-0:amd64.
(Reading database ... 346483 files and directories currently installed.)
Preparing to unpack libpng12-0_1.2.54-1ubuntu1_amd64-patched.deb ...
Unpacking libpng12-0:amd64 (1.2.54-1ubuntu1) ...
Setting up libpng12-0:amd64 (1.2.54-1ubuntu1) ...
Processing triggers for libc-bin (2.30-0ubuntu3) ...
```

If you want to look at what is inside the package you can do the following:
```
$ dpkg-deb -R libpng12-0_1.2.54-1ubuntu1_amd64.deb tmp
$ cd tmp
$ find
.
./usr
./usr/lib
./usr/lib/x86_64-linux-gnu
./usr/lib/x86_64-linux-gnu/libpng12.so.0
./usr/share
./usr/share/doc
./usr/share/doc/libpng12-0
./usr/share/doc/libpng12-0/libpng-1.2.54.txt.gz
./usr/share/doc/libpng12-0/README.gz
./usr/share/doc/libpng12-0/KNOWNBUG
./usr/share/doc/libpng12-0/README.Debian
./usr/share/doc/libpng12-0/TODO
./usr/share/doc/libpng12-0/ANNOUNCE
./usr/share/doc/libpng12-0/copyright
./usr/share/doc/libpng12-0/changelog.Debian.gz
./lib
./lib/x86_64-linux-gnu
./lib/x86_64-linux-gnu/libpng12.so.0
./lib/x86_64-linux-gnu/libpng12.so.0.54.0
./DEBIAN
./DEBIAN/control
./DEBIAN/shlibs
./DEBIAN/triggers
./DEBIAN/md5sums
$ ls -gG lib/x86_64-linux-gnu
total 148
lrwxrwxrwx 1     18 Jan  7  2016 libpng12.so.0 -> libpng12.so.0.54.0
-rw-r--r-- 1 149904 Jan  7  2016 libpng12.so.0.54.0
$ ls -gG usr/lib/x86_64-linux-gnu
total 0
lrwxrwxrwx 1 35 Jan  7  2016 libpng12.so.0 -> /lib/x86_64-linux-gnu/libpng12.so.0

```
The original error from above was `unable to install new version of '/lib/x86_64-linux-gnu/libpng12.so.0': No such file or directory`.
So from here you can fix the symbolic links and or move the files to the correct directory. Of course you could manually move the files to their final destination instead of editing their package.  The problem with this is that some programs like tizen studio use dpkg to see if you have a library installed instead of checking to see if the library exists.  So in that case you must install the package.