---
layout: post
title:  "Qt deploy on Linux"
date:   2019-07-11 17:43:05 +0800
categories: C++ Qt
---

# ls
```
Hoba  Hoba.sh  lib  lnd  plugins  qml  qt.conf
```

## Hoba.sh
```
#!/bin/sh
appname=`basename $0 | sed s,\.sh$,,`

dirname=`dirname $0`
tmp="${dirname#?}"

if [ "${dirname%$tmp}" != "/" ]; then
dirname=$PWD/$dirname
fi
LD_LIBRARY_PATH=$dirname/lib
export LD_LIBRARY_PATH
$dirname/$appname "$@"
```

## qt.conf
```
[Paths]
Prefix = ./
Plugins = plugins
Imports = qml
Qml2Imports = qml
```

## plugins/
```
bearer  iconengines  imageformats  platforminputcontexts  platforms  xcbglintegrations

(我是用的linuxdeployqt,生成的。从github上下载源码编译安装。linuxdeployqt Hoab -qml=../../src/pages)
```

## qml
```
com  QtGraphicalEffects  QtQml  QtQuick  QtQuick.2
(com 是生成二维码的一个工具用到的import模块,https://github.com/toby20130333/qtquickqrencode.git)
```

## lib
```
fonts             libgrpc++.so.1     libQt5DBus.so.5     libQt5QuickControls2.so.5   libQt5Svg.so.5              libssl.so.1.1
libcrypto.so.1.1  libgrpc.so.7       libQt5Gui.so.5      libQt5QuickParticles.so.5   libQt5VirtualKeyboard.so.5  libxcb-xinerama.so.0
libgrpc.so        libprotobuf.so.19  libQt5Network.so.5  libQt5Quick.so.5            libQt5Widgets.so.5
libgrpc++.so      libQt5Core.so.5    libQt5Qml.so.5      libQt5QuickTemplates2.so.5  libQt5XcbQpa.so.5

(fonts 存放的是字体，里面存了个微软亚黑的.ttf)
```

## 启动Hoba.sh时查看调用过程
```
QT_DEBUG_PLUGINS=1 ./Hoba.sh
```

