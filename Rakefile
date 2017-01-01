#
# Copyright (c) 2008-2017 the Urho3D project.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
#

task default: %w[sysroot_update]

# Usage: NOT intended to be used manually
desc 'Update arm64-sysroot in the master branch'
task :sysroot_update do
  system 'sudo apt-get update -q -y && sudo apt-get install -q -y --no-install-recommends kpartx qemu binfmt-support qemu-user-static' or abort 'Failed to install dependencies'
  system 'git clone --depth=1 https://github.com/urho3d/arm64-sysroot.git' or abort 'Failed to clone existing sysroot'
  system 'wget https://cloud-images.ubuntu.com/$name/current/$name-server-cloudimg-arm64-root.tar.gz -O root.tar.gz' or abort 'Failed to download latest arm64 root tarball'
  system 'echo Untarring... && sudo tar xf root.tar.gz -C arm64-sysroot && sudo chown -R $USER: arm64-sysroot && rm root.tar.gz' or abort 'Failed to untar arm64 root tarball'
  system 'echo Installing... && cp /usr/bin/qemu-aarch64-static arm64-sysroot/usr/bin && for f in dev dev/pts dev/shm proc run sys; do sudo mount --bind /$f arm64-sysroot/$f; done && sudo chroot arm64-sysroot /bin/bash -c \'apt-get update && apt-get -q -y upgrade && apt-get -q -y install libgles{1,2}-mesa-dev libx11-dev libxcursor-dev libxext-dev libxi-dev libxinerama-dev libxrandr-dev libxrender-dev libxss-dev libxxf86vm-dev libasound2-dev libpulse-dev libdbus-1-dev libreadline6-dev libudev-dev\' && for f in dev/pts dev/shm dev proc run sys; do sudo umount arm64-sysroot/$f; done' or abort 'Failed to install prerequisite software packages in new sysroot'
  system 'sudo chown -R $USER: arm64-sysroot' or abort 'Failed to change file ownership'
  `find arm64-sysroot/usr/lib/aarch64-linux-gnu -type l |xargs ls -l |grep ' /'`.split("\n").each { |l| link = / ([^ ]+) -> ([^ ]+)/.match l; system "ln -sf #{link[2].sub '/', '../../../'} #{link[1]}" } and system 'ln -sfr arm64-sysroot/usr/lib/aarch64-linux-gnu/libGLESv1_CM.so arm64-sysroot/usr/lib/aarch64-linux-gnu/libGLESv1_CM.so.1 && ln -sfr arm64-sysroot/usr/lib/aarch64-linux-gnu/libGLESv2.so arm64-sysroot/usr/lib/aarch64-linux-gnu/libGLESv2.so.2' or abort 'Failed to fix symbolic links'
  replaced_content = File.read('arm64-sysroot/README.md').gsub(/\(.*?\)/m, "(#{File.new('arm64-sysroot/usr').mtime.utc.to_s})") and File.open('arm64-sysroot/README.md', 'w') { |file| file.puts replaced_content }
  system "echo Committing... && cd arm64-sysroot && git config user.name '#{ENV['GIT_NAME']}' && git config user.email '#{ENV['GIT_EMAIL']}' && git remote set-url --push origin https://#{ENV['GH_TOKEN']}@github.com/urho3d/arm64-sysroot.git && git add -Af . && rm -rf * && git checkout -- README.md usr/include usr/lib lib && git add -A . && ( git commit -q -m \"Travis CI: arm64 sysroot update at #{Time.now.utc}.\" || true) && git push -q >/dev/null 2>&1" or abort 'Failed to push new sysroot'
end
