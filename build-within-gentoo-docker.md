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


