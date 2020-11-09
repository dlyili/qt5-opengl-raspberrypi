# Build notes for qtcreator 4.13.2
These are notes for a native build of QtCreator 4.13.2 on a Raspberry Pi 4, 4GB ram, running Raspbian Buster. 

## Install Instructions
See [Qt5.15 Install Instructions](https://github.com/koendv/qt5-opengl-raspberrypi/blob/master/README.md#install-instructions).

## Prerequisites

Beginning with a clean [2019-09-26-raspbian-buster-lite](https://www.raspberrypi.org/downloads/raspbian/)

```
wget https://github.com/koendv/qt5-opengl-raspberrypi/releases/download/v5.15.5-1/qt5-opengl-dev_5.15.1_armhf.deb
apt-get update
apt install ./qt5-opengl-dev_5.15.1_armhf.deb
apt-get install build-essential qtchooser
```
## Build qtcreator
```
wget -c http://download.qt.io/official_releases/qtcreator/4.13/4.13.2/qt-creator-opensource-src-4.13.2.tar.xz
tar xf qt-creator-opensource-src-4.13.2.tar.xz 
mkdir  build-qt-creator
cd build-qt-creator/
export QT_SELECT=qt5-opengl
export PATH=/usr/lib/qt5.15/bin:$PATH
qmake ../qt-creator-opensource-src-4.13.2
make
```
Compile time is less than six hours.

## Create debian package
```
mkdir deb
INSTALL_ROOT=$PWD/deb/usr/lib/qt5.15 make install
mkdir deb/DEBIAN
cat > deb/DEBIAN/control <<EOD
Package: qt5-opengl-qtcreator
Version: 4.13.2
Maintainer: Koen <koen@mcvax.org>
Priority: optional
Section: libs
Bugs: https://github.com/koendv/qt5-opengl-raspberrypi/issues
Homepage: https://github.com/koendv/qt5-opengl-raspberrypi
Depends: build-essential
Architecture: armhf
Description: qtcreator IDE for Qt5.15 LTS with desktop OpenGL
 qtcreator IDE for Qt5.15 LTS, compiled for raspberry pi 4 running 2019-09-26-raspbian-buster[-lite.].
 Install qt5-opengl-dev_5.15.1_armhf.deb first.
EOD
fakeroot dpkg-deb -b ./deb/ .
```
 This produces the debian package file ```qt5-opengl-qtcreator_4.13.2_armhf.deb```
 
 This completes the build notes.




