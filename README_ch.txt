mkdir ~/qt-build
cd ~/qt-build
#安装qt 到 /opt/qtpi
~/qt-source/configure \
-prefix /opt/qtpi \
-opensource \
-confirm-license \
-nomake examples \
-skip qtimageformats

make -j4
make install

查看树莓派系统信息
lsb_release  -a
cat /proc/cpuinfo

1.参考网址:
https://github.com/koendv/qt5-opengl-raspberrypi
https://github.com/oniongarlic/qt-raspberrypi-configuration.git 
https://github.com/Devin-Cooper/qt-raspberrypi-configuration

编译教程
https://www.tal.org/tutorials/building-qt-512-raspberry-pi
https://www.tal.org/tutorials/building-qtcreator-raspberry-pi-debian-stretch

按照koendv按照步骤编译,可以下载其编译好的库:https://github.com/koendv/qt5-opengl-raspberrypi/releases

2.编译
2.1基础依赖安装
apt-get update
apt-get install libgl1-mesa-dev libglu1-mesa-dev mesa-common-dev
apt-get install build-essential git libfontconfig1-dev libdbus-1-dev libfreetype6-dev libicu-dev libinput-dev libxkbcommon-dev libsqlite3-dev libssl-dev libpng-dev libjpeg-dev libglib2.0-dev libraspberrypi-dev libcups2-dev libasound2-dev
apt-get install libfontconfig1-dev libfreetype6-dev libx11-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libx11-xcb-dev libxcb-glx0-dev libxkbcommon-x11-dev libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync0-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libnss3-dev libxcomposite-dev libxcursor-dev libxtst-dev libxrandr-dev gperf bison flex ninja-build 

apt-get install libclang-dev
sudo apt-get install "^libxcb.*" libx11-xcb-dev libglu1-mesa-dev libxrender-dev
apt install libbluetooth-dev libpq-dev libmariadbclient-dev libcups2-dev

2.2下载qt源码
cd /opt
wget -c http://download.qt-project.org/official_releases/qt/5.15/5.15.1/single/qt-everywhere-src-5.15.1.tar.xz
tar xf qt-everywhere-src-5.15.1.tar.xz

修改两个源码:
/opt/qt-everywhere-src-5.15.1/qtbase/src/plugins/platforms/xcb/gl_integrations/xcb_egl/qxcbeglwindow.cpp
/opt/qt-everywhere-src-5.15.1/qtscript/src/3rdparty/javascriptcore/JavaScriptCore/wtf/Platform.h

2.3下载qt-raspberrypi-configuration
git clone https://github.com/oniongarlic/qt-raspberrypi-configuration.git
cd qt-raspberrypi-configuration
make install DESTDIR=../qt-everywhere-src-5.15.1
cd ..

2.4编译
mkdir build-qt
cd build-qt
配置:
PKG_CONFIG_LIBDIR=/usr/lib/arm-linux-gnueabihf/pkgconfig:/usr/share/pkgconfig \
../qt-everywhere-src-5.15.1/configure -platform linux-rpi3-g++ \
-v \
-opengl desktop -eglfs \
-no-gtk \
-opensource -confirm-license -release \
-reduce-exports \
-force-pkg-config \
-nomake examples -no-compile-examples \
-skip qtwayland \
-no-feature-geoservices_mapboxgl \
-qt-pcre \
-no-pch \
-ssl \
-evdev \
-system-freetype \
-fontconfig \
-glib \
-prefix /usr/lib/qt5.15  \
-qpa eglfs

检查:/opt/build-qt/config.summary文件，查看Support qpa-xcb 是否为true,如果不是，需要
删除:/opt/build-qt/config.* Makefile等几个文件，再次尝试。(可能libxcb没装好)

修改"/opt/build-qt/qtwebengine/src/core/Makefile.gn_run"文件，搜索“ninja -v”，然后修改
为:"ninja -j2 -v",这用于减少虚拟内存，否则qtwebengine编译会因为内存不足而失败

make

2.5创建debian package
2.5.1
mkdir deb
INSTALL_ROOT=$PWD/deb make install

2.5.2
mkdir -p deb/usr/share/qtchooser/
cat > deb/usr/share/qtchooser/qt5-opengl.conf <<EOD
/usr/lib/qt5.15/bin/
/usr/lib/qt5.15/lib/
EOD

2.5.3
mkdir deb/DEBIAN
cat > deb/DEBIAN/control <<EOD
Package: qt5-opengl-dev
Version: 5.15.1
Maintainer: Koen <koen@mcvax.org> jjj <jiangjianjun716@163.com
Priority: optional
Section: libs
Bugs: https://github.com/koendv/qt5-opengl-raspberrypi/issues
Homepage: https://github.com/dlyili/qt5-opengl-raspberrypi
Depends: libgl1-mesa-dev, libglu1-mesa-dev, mesa-common-dev, libfontconfig1-dev, libdbus-1-dev, libfreetype6-dev, libicu-dev, libinput-dev, libxkbcommon-dev, libsqlite3-dev, libssl-dev, libpng-dev, libjpeg-dev, libglib2.0-dev, libraspberrypi-dev, libcups2-dev, libasound2-dev, libfontconfig1-dev, libfreetype6-dev, libx11-dev, libxext-dev, libxfixes-dev, libxi-dev, libxrender-dev, libxcb1-dev, libx11-xcb-dev, libxcb-glx0-dev, libxkbcommon-x11-dev, libxcb-keysyms1-dev, libxcb-image0-dev, libxcb-shm0-dev, libxcb-icccm4-dev, libxcb-sync0-dev, libxcb-xfixes0-dev, libxcb-shape0-dev, libxcb-randr0-dev, libxcb-render-util0-dev, libnss3-dev, libxcomposite-dev, libxcursor-dev, libxtst-dev, libxrandr-dev, gperf, bison, flex, ninja-build
Architecture: armhf
Description: Qt5.15 LTS with desktop OpenGL
 Qt5.15 LTS "long term support" with desktop OpenGL,
 compiled for raspberry pi 4 running 2019-09-26-raspbian-buster[-lite.]
 The package is suitable for compiling desktop-style, windowed Qt apps under X11. The OpenGL support is in software, using Mesa.
EOD

fakeroot dpkg-deb -b ./deb/ .

3.qt creator
wget -c http://download.qt.io/official_releases/qtcreator/4.13/4.13.2/qt-creator-opensource-src-4.13.2.tar.xz
tar xf qt-creator-opensource-src-4.13.2.tar.xz 
mkdir  build-qt-creator
cd build-qt-creator/
export QT_SELECT=qt5-opengl
export PATH=/usr/lib/qt5.15/bin:$PATH
qmake ../qt-creator-opensource-src-4.13.2
make