## Budiling a CoreOS image using theGentoo Docker at the CoreOS

### Gentoo Docker
* Docker Hub :  https://hub.docker.com/r/gentoo
* Gentoo Stage 3 docker - https://hub.docker.com/r/gentoo/stage3-amd64/
* Gentoo Portage docker - https://hub.docker.com/r/gentoo/portage/

### Run Gentoo Docker
* run the gentoo/portage as the data volume docker and run the gentoo/stage3-amd64 docker with the volume /usr/portage
* commit the docker as the new gentoo:iamyaw image
```
  core@ctestdocker002 $ docker create -v /usr/portage --name gentoo-portage gentoo/portage
  core@ctestdocker002 $ docker run -ti --name gentoo-stage3 --volumes-from gentoo-portage gentoo/stage3-amd64
  core@ctestdocker002 $ docker commit gentoo-stage3 gentoo:iamyaw 
  core@ctestdocker002 $ docker run -ti --name gentoo-iamyaw --volumes-from gentoo-portage gentoo:iamyaw
```
```
  sh-4.3# ls /usr/portage
```

### Create the working user 'core'
* Add the group core as gid 500 and the user core as uid 500
* gid and uid must be 500, group and user name must be core
* the user core also belongs to wheel group for sudo
```
  sh-4.3# groupadd -g 500 core
  sh-4.3# useradd -m -u 500 -g 500 -G wheel core
```

### Installing some tools into the gentoo docker
* use emerge tool to install something in the Gentoo linux
```
  sh-4.3# ls -d /usr/portage/*/sudo
  /usr/portage/app-admin/sudo
  sh-4.3# emerge install app-admin/sudo
  sh-4.3# ls -d /usr/portage/*/vim
  /usr/portage/app-editors/vim  /usr/portage/licenses/vim
  sh-4.3# emerge install app-editors/vim
```
```
  core@ctestdocker002 $ docker commit gentoo-iamyaw gentoo:iamyaw
```
