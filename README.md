# strongswan on windows
关于strongswan在windows平台上面的尝试

结论：官网介绍：不支持虚拟ip,所以不能用做客户端。

【这条路不通】

以下是踩坑过程：
1、build openssl
一定要用tar的命令行解压源码压缩包，要不然各种报错。
./Configure --prefix=$PWD/../libs no-idea no-mdc2 no-rc5 shared mingw
./Configure --prefix=$PWD/../libs no-idea no-mdc2 no-rc5 shared mingw64

make depend

make && make install

2020.02.28测试成功.



2、build strongswan

CC=x86_64-w64-mingw32-gcc 

CFLAGS="-g -O2 -Wall -Wno-format -Wno-format-security -Wno-pointer-sign -Werror -I/home/Administrator/work/myvpn/libs/include -mno-ms-bitfields" 

LDFLAGS="-L/home/Administrator/work/myvpn/libs/lib"



./configure --prefix=$PWD/../libs --host=x86_64-w64-mingw32  --disable-defaults --enable-monolithic --enable-static --enable-svc --enable-ikev2 --enable-ikev1 --enable-nonce --enable-pem --enable-pkcs1 --enable-x509 --enable-openssl --enable-socket-win --enable-kernel-wfp --enable-kernel-iph --enable-pubkey --enable-swanctl  --enable-libipsec  --enable-eap-tls  --enable-eap-peap --enable-eap-gtc --enable-eap-dynamic --enable-eap-identity --enable-eap-mschapv2 --enable-md4 --enable-ipseckey --enable-dnscert --enable-files --enable-sha3 host_alias=x86_64-w64-mingw32 CC=x86_64-w64-mingw32-gcc --with-swanctldir=swanctl --with-strongswan-conf=strongswan.conf


make -j4
make check


libstrongswan: sha3 md4 nonce x509 pubkey pkcs1 pem openssl files
libcharon:     dnscert ipseckey kernel-wfp kernel-iph socket-win vici eap-identity eap-gtc eap-mschapv2 eap-dynamic eap-tls eap-peap
libtnccs:
libtpmtss:



2020年3月4日测试可以启动守护程序。

在kernel_wfp_ipsec.c中register_events时需要启动相关服务，加入以下代码，否则报
registering for WFP events failed错误导致后面报socket找不到依赖的问题。
FWP_VALUE0 val = {0};
val.type = FWP_UINT32;
val.uint32 = 1;
res = FwpmEngineSetOption0(this->handle, FWPM_ENGINE_COLLECT_NET_EVENTS, &val);
if (res != ERROR_SUCCESS)
{
    DBG1(DBG_KNL, "registering for WFP SetOption failed: 0x%08x", res);
}



#必须关闭windows的IKEEXT（IKE and AuthIP IPsec Keying Modules）服务
要不然端口被占用，接收不了数据。
#执行命令
先启动守护程序charon-svc.exe

#加载配置

swanctl -q

swanctl --initiate --child ios_ikev2




