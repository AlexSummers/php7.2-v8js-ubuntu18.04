# php7.2-v8js-ubuntu18.04
install v8js at Ubuntu 18.04

# Install required dependencies
sudo apt-get install build-essential curl git python libglib2.0-dev

cd /tmp

# Install depot_tools first (needed for source checkout)
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH=`pwd`/depot_tools:"$PATH"

# Download v8
fetch v8
cd v8

# (optional) If you'd like to build a certain version:
git checkout 6.4.388.18
gclient sync

# Setup GN
tools/dev/v8gen.py -vv x64.release -- is_component_build=true

# Build
ninja -C out.gn/x64.release/

# Install to /opt/v8/
sudo mkdir -p /opt/v8/{lib,include}
sudo cp out.gn/x64.release/lib*.so out.gn/x64.release/*_blob.bin \
  out.gn/x64.release/icudtl.dat /opt/v8/lib/
sudo cp -R include/* /opt/v8/include/

# Compile php-v8js itself
cd /tmp
git clone https://github.com/phpv8/v8js.git
cd v8js
phpize

#libv8 fix
sudo cp out.gn/x64.release/lib*.so /usr/lib/x86_64-linux-gnu/

./configure --with-v8js=/opt/v8 LDFLAGS="-lstdc++"
sudo make
sudo make install
