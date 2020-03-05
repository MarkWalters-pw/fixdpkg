# fixdpkg
Extracts a debian package, runs your custom script on it, then repackages it.


patch-libpng12 is an example of what you can do with fixdpkg.  It will allow you to install libpng12 on newer versions of Ubuntu.

This is the error you get when trying to install libpng12 on Ubuntu focal.
```
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