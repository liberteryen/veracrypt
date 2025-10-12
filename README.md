# This repository is a fork of https://github.com/veracrypt/VeraCrypt/ containing fixes for various issues found in the original.
```bash
wget https://github.com/wxWidgets/wxWidgets/releases/download/v3.3.0/wxWidgets-3.3.0.tar.bz2
tar -xvf wxWidgets-3.3.0.tar.bz2
```

The following line in the Makefile:
```bash
WX_CONFIGURE_FLAGS += --enable-unicode --disable-shared --disable-dependency-tracking --enable-exceptions --enable-std_string --enable-dataobj --enable-mimetype
```
was changed to:
```bash
WX_CONFIGURE_FLAGS += --disable-shared --disable-dependency-tracking --enable-exceptions --enable-dataobj --enable-mimetype  
```
The ./Crypto/Argon2/src/blake2/blake2b.h file was copied into the ./Crypto directory.

The compilation was started with the following command:
```bash
make NOGUI=1 WXSTATIC=1 WX_ROOT=/XXXX/wxWidgets-3.3.0 wxbuild

make NOGUI=1 WXSTATIC=1 WX_ROOT=/XXXX/wxWidgets-3.3.0 CC=gcc -j16  
```
```bash
apk add build-base pkgconf linux-headers libusb-dev pcsc-lite-dev eudev-dev
```
