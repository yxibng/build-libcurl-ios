# build curl for macOS

1. 需要编译`curl + openssl`的组合
2.   unisocket 使用的是`boringssl`, 在同一个项目中不能同时存在`openssl`和`boringssl `所以 编译 `curl + boringssl`的组合.

## 步骤：

1. 编译[unisocket](https://team.duobeiyun.com/diffusion/273/) , 得到`libcrypto.a`,`libssl.a`

	```
	➜  libuniversaltransport git:(master) ✗ tree out -L 2
	out
	├── include
	│   ├── IUniversalSocket.h
	│   ├── UniSocketBuildConfig.h
	│   ├── UniversalSocket.h
	│   ├── UniversalSocketBackend.h
	│   ├── UniversalSocketFactory.h
	│   ├── uv
	│   └── uv.h
	└── lib
	    ├── libMattClient.a
	    ├── libQuicClient.a
	    ├── libUniSocket.a
	    ├── libcrypto.a
	    ├── liblsquic.a
	    ├── libprotobuf-lite.a
	    ├── libprotobuf.a
	    ├── libssl.a
	    ├── libuuid.a
	    ├── libuv.a
	    └── libuv_a.a
	```

2. 编译`curl+ boringssl `
	
	需要连接boringssl 的 `include `, `lib`

	```
	boringssl
	├── include
	│   └── openssl
	└── lib
	    ├── libcrypto.a
	    └── libssl.a
	```
	
	
	- `include` 头文件: 将`libuniversaltransport/3rdparty/archive/boringssl.zip`解压得到`boringssl/include`文件夹。
	- `lib` 编译`unisocket ` 得到的`libcrypto.a`,`libssl.a`
	
	
	脚本 `build_curl_macos.sh`
	
	```
	#!/bin/sh
	
	
	# 将$DIR替换
	export CPPFLAGS="-I$DIR/boringssl/include"
	export LDFLAGS="-L$DIR/boringssl/lib" 
	
	
	if [  -f "./out" ]; then
	    rm -rf out
	fi
	
	mdkir -p out 
	
	./configure \
	--without-libidn2 \
	--without-librtmp \
	--disable-ldap \
	--without-libpsl \
	--disable-shared \
	--enable-static \
	--prefix=`pwd`/out
	
	make -j `nproc`
	make install 
	
	open out 
	```	 
	
	# 使用系统的 secure—transport 来编译
	```bash
	#!/bin/sh
    if [  -f "./out" ]; then
	    rm -rf out
	fi
	
    export MACOSX_DEPLOYMENT_TARGET="10.10"

	mdkir -p out 	
    ./configure \
    --with-secure-transport \
	--without-libidn2 \
	--without-librtmp \
	--disable-ldap \
	--without-libpsl \
	--disable-shared \
	--enable-static \
    --without-ssl \
	--enable-symbol-hiding \
	--prefix=`pwd`/out

    make -j `nproc`
	make install 
	
	open out 
	```