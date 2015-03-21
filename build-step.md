
git config --global user.email "you@example.com"
git config --global user.name "Your Name"

git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="$PATH":`pwd`/depot_tools

mkdir coreos; cd coreos

repo init -u https://github.com/coreos/manifest.git -g minilayout --repo-url https://chromium.googlesource.com/external/repo.git

repo sync

./chromite/bin/cros_sdk

./set_shared_user_password.sh

echo amd64-usr > .default_board

./setup_board

./build_packages

./build_image --noenable_rootfs_verification dev
./build_image prod --group production

image_to_vm.sh
./image_to_vm.sh --from=../build/images/amd64-usr/developer-602.0.0+2015-02-24-1954-a1 --board=amd64-usr

./coreos_developer_qemu.sh -curses
./coreos_developer_qemu.sh -a ~/.ssh/authorized_keys -p 2223 -- -curses


src/third_party/portage-stable/profiles/uclibc/make.defaults
src/third_party/coreos-overlay/profiles/coreos/targets/generic/make.defaults
src/third_party/coreos-overlay/profiles/coreos/targets/generic/package.use
src/third_party/coreos-overlay/profiles/coreos/targets/sdk/make.defaults

http://www.slideshare.net/higebu/building-andcustomizingcoreos
