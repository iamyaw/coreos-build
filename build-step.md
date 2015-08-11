### ChromeOS depot_tools
* git config --global user.email "you@example.com"
* git config --global user.name "Your Name"
* git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
* export PATH="$PATH":\`pwd\`/depot_tools

### Repo Sync
* mkdir coreos; cd coreos
* repo init -u https://github.com/coreos/manifest.git -g minilayout --repo-url  https://chromium.googlesource.com/external/repo.git
* repo sync

### CoreOS SDK
* ./chromite/bin/cros_sdk
* ./set_shared_user_password.sh

### Build Package
* echo amd64-usr > .default_board
* ./setup_board
* ./build_packages

### Build Image
* ./build_image --noenable_rootfs_verification dev
* ./build_image prod --group production
* image_to_vm.sh
* ./image_to_vm.sh --from=../build/images/amd64-usr/developer-602.0.0+2015-02-24-1954-a1 --board=amd64-usr --format=virtualbox

### qemu
* ./coreos_developer_qemu.sh -curses
* ./coreos_developer_qemu.sh -a ~/.ssh/authorized_keys -p 2223 -- -curses

## Custom Overlay
* src/third_party/portage-stable/profiles/uclibc/make.defaults
* src/third_party/coreos-overlay/profiles/coreos/targets/generic/make.defaults
```
    repo diff
    project src/third_party/coreos-overlay/
    diff --git a/profiles/coreos/targets/generic/make.defaults b/profiles/coreos/targets/g
    index 27f71ff..2dd5720 100644
    --- a/profiles/coreos/targets/generic/make.defaults
    +++ b/profiles/coreos/targets/generic/make.defaults
    @@ -1,7 +1,7 @@
    -USE="cros-debug acpi usb symlink-usr cryptsetup policykit -pam"
    +USE="cros-debug acpi usb symlink-usr cryptsetup policykit pam"
```
* src/third_party/coreos-overlay/profiles/coreos/targets/generic/package.use
* ~~src/third_party/coreos-overlay/profiles/coreos/targets/sdk/make.defaults
* src/third_party/portage-stable/profiles/default/linux/make.defaults
```
    project src/third_party/portage-stable/
    diff --git a/profiles/default/linux/make.defaults b/profiles/default/linux/make.
    index dfa4843..e52ca1b 100644
    --- a/profiles/default/linux/make.defaults
    +++ b/profiles/default/linux/make.defaults
    @@ -12,7 +12,7 @@
    -USE="berkdb crypt ipv6 ncurses nls pam readline ssl tcpd zlib"
    +USE="berkdb crypt ipv6 ncurses nls pam readline ssl tcpd zlib kerberos"
```
* src/third_party/portage-stable/profiles/default/linux/package.use

## Bug fix

* scripts/build_packages
`# util-linux[udev] -> virtual->udev -> systemd -> util-linux`
`break_dep_loop sys-apps/util-linux udev,systemd sys-apps/systemd cryptsetup`

### Links
* http://www.slideshare.net/higebu/building-andcustomizingcoreos
* https://chromium.googlesource.com/chromiumos/overlays/
* https://chromium.googlesource.com/?format=HTML
* http://www.chromium.org/chromium-os/build
* https://www.chromium.org/developers/how-tos/build-instructions-chromeos
* http://www.chromium.org/chromium-os/how-tos-and-troubleshooting/portage-build-faq
* https://wiki.gentoo.org/wiki/Handbook:X86
