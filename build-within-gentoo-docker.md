## Budiling a CoreOS image using theGentoo Docker at the CoreOS

### Gentoo Docker
* Docker Hub :  https://hub.docker.com/r/gentoo
* Gentoo Stage 3 docker - https://hub.docker.com/r/gentoo/stage3-amd64/
* Gentoo Portage docker - https://hub.docker.com/r/gentoo/portage/

### Run Gentoo Docker
* Run the gentoo/portage as the data volume docker and run the gentoo/stage3-amd64 docker with the volume /usr/portage
* Commit the docker as the new gentoo:iamyaw image
 * In order to run coreos SDK, /sys/fs/cgroup must exists so that the docker must be run as privileged mode with binding /sys/fs/cgroup. As binding /sys/fs/cgroup, the only readonly mode works. Use '-v /sys/fs/cgroup:/sys/fs/cgroup:ro'.
```
  core@ctestdocker002 ~ $ docker create -v /usr/portage --name gentoo-portage gentoo/portage
  core@ctestdocker002 ~ $ docker run -ti --name gentoo-stage3 --volumes-from gentoo-portage gentoo/stage3-amd64
  core@ctestdocker002 ~ $ docker commit gentoo-stage3 gentoo:iamyaw 
  core@ctestdocker002 ~ $ docker run -ti --privileged -v /sys/fs/cgroup:/sys/fs/cgroup:ro --name gentoo-iamyaw --volumes-from gentoo-portage gentoo:iamyaw
  sh-4.3# ls /usr/portage
```
 * Also the CoreOS SDK requires fuse module too. Execute 'sudo modprobe fuse'
```
  core@ctestdocker002 ~ $ sudo modprobe -v fuse
```

### Create the working user 'core'
* Add the group core as gid 500 and the user core as uid 500
 * gid and uid must be 500, group and user name must be core
 * The user core also belongs to wheel group for sudo
```
  sh-4.3# groupadd -g 500 core
  sh-4.3# useradd -m -u 500 -g 500 -G wheel core
```

### Installing some tools into the gentoo docker
* Use emerge tool to install something in the Gentoo linux
 * The Gentoo linux does not have a package tool. Instead, should compile from the source. So it is called as 'merging'.
```
  sh-4.3# ls -d /usr/portage/*/sudo
  /usr/portage/app-admin/sudo
  sh-4.3# emerge app-admin/sudo
  sh-4.3# ls -d /usr/portage/*/vim
  /usr/portage/app-editors/vim  /usr/portage/licenses/vim
  sh-4.3# emerge app-editors/vim
  ...
  sh-4.3# emerge dev-vcs/git
  sh-4.3# emerge net-misc/curl
```
* Periodically commit the running docker
```
  core@ctestdocker002 ~ $ docker commit gentoo-iamyaw gentoo:iamyaw
```

### Prepare CoreOS build environment
* Refer to build steps in https://github.com/iamyaw/coreos-build/blob/master/build-step.md
* ChromeOS depot_tools
```
  sh-4.3# sudo su - core
  core@0a829090297a ~ $ git config --global user.email "iamyaw@gmail.com"
  core@0a829090297a ~ $ git config --global user.name "iamyaw"
  core@0a829090297a ~ $ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
  core@0a829090297a ~ $ export PATH="$PATH":`pwd`/depot_tools
```
 * Edit .bashrc of the user core
```
  # /etc/skel/.bashrc
  export PATH="$PATH":$HOME/depot_tools
```
* Repo Sync
```
  core@0a829090297a ~/core $ mkdir coreos; cd coreos
  core@0a829090297a ~/coreos $ repo init -u https://github.com/coreos/manifest.git -g minilayout --repo-url https://chromium.googlesource.com/external/repo.git
  core@0a829090297a ~/coreos $ repo sync
```
* CoreOS SDK
 * The CoreOS SDK downloads and unpack the ChromeOS, which is based on the Gentoo Linux. "Unpacking STAGE3..."
 * The 'set_shared_user_passwd.sh' is important because when it is the only way to login into the build CoreOS image.
 * Use '4un4mecore' as the password
```
 core@0794005b3bf1 ~/coreos $ ./chromite/bin/cros_sdk
 core@0794005b3bf1 ~/trunk/src/scripts $ ./set_shared_user_password.sh
 Enter password for shared user account: Password set in /etc/shared_user_passwd.txt
```
* Build Package
```
  core@0794005b3bf1 ~/trunk/src/scripts $ echo "amd64-usr" > .default_board
  core@0794005b3bf1 ~/trunk/src/scripts $ ./setup_board
  core@0794005b3bf1 ~/trunk/src/scripts $ ./build_packages
```
* Build Image
 * building production image
```
  core@0794005b3bf1 ~/trunk/src/scripts $ ./build_image prod --group production
  core@0794005b3bf1 ~/trunk/src/scripts $ image_to_vm.sh
  core@0794005b3bf1 ~/trunk/src/scripts $ ./image_to_vm.sh --from=../build/images/amd64-usr/developer-602.0.0+2015-02-24-1954-a1 --board=amd64-usr --format=virtualbox
```
  * building dev image
```
  core@0794005b3bf1 ~/trunk/src/scripts $./build_image --noenable_rootfs_verification dev
```
* qemu
```
  core@0794005b3bf1 ~/trunk/src/scripts $ ./coreos_developer_qemu.sh -curses
  core@0794005b3bf1 ~/trunk/src/scripts $ ./coreos_developer_qemu.sh -a ~/.ssh/authorized_keys -p 2223 -- -curses
```

### Custom Overlay
* Change coreos-overlay in order to build the custom CoreOS image. e.g. including Kerberos
* PAM module
 * Edit the file 'src/third_party/coreos-overlay/profiles/coreos/targets/generic/make.defaults'
 * Notice that it is outside 'cros_sdk'
 * Related files :
   * src/third_party/portage-stable/profiles/uclibc/make.defaults
```
  core@0794005b3bf1 ~/coreos $ repo diff
  
  project src/third_party/coreos-overlay/
  diff --git a/profiles/coreos/targets/generic/make.defaults b/profiles/coreos/ta
  index 4238a59..2f5c6b2 100644
  --- a/profiles/coreos/targets/generic/make.defaults
  +++ b/profiles/coreos/targets/generic/make.defaults
  @@ -1,7 +1,7 @@
   # Copyright (c) 2012 The Chromium OS Authors. All rights reserved.
   # Distributed under the terms of the GNU General Public License v2
  
  -USE="cros-debug acpi usb symlink-usr cryptsetup policykit -pam"
  +USE="cros-debug acpi usb symlink-usr cryptsetup policykit pam"
   USE="${USE} -cros_host -expat -cairo -X -man"
   USE="${USE} -acl -cracklib -gpm -python -sha512"
   USE="${USE} -fortran -abiword -perl -cups -poppler-data -nls"
```
* Kerberos module
 * Edit the file 'src/third_party/portage-stable/profiles/default/linux/make.defaults'
 * Related files :
   * src/third_party/coreos-overlay/profiles/coreos/targets/generic/package.use
   * src/third_party/coreos-overlay/profiles/coreos/targets/sdk/make.defaults
   * src/third_party/portage-stable/profiles/default/linux/package.use
```
  core@0794005b3bf1 ~/coreos $ repo diff

  project src/third_party/portage-stable/
  diff --git a/profiles/default/linux/make.defaults b/profiles/default/linux/make
  index dfa4843..e52ca1b 100644
  --- a/profiles/default/linux/make.defaults
  +++ b/profiles/default/linux/make.defaults
  @@ -12,7 +12,7 @@
  
  
   # Default starting set of USE flags for all default/linux profiles.
  -USE="berkdb crypt ipv6 ncurses nls pam readline ssl tcpd zlib"
  +USE="berkdb crypt ipv6 ncurses nls pam readline ssl tcpd zlib kerberos"
  
   # make sure toolchain has sane defaults <tooclhain@gentoo.org>
   USE="${USE} fortran openmp"
```
* Build Package
 * Notice that it is inside 'cros_sdk'
```
  core@0794005b3bf1 ~/trunk/src/scripts $ ./build_packages
```

