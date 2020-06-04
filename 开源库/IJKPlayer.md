# IJKPlayer

哔哩哔哩开源的视频播放库。

### 编译

1. 编译ffmpeg报错
	
	源码编译时，因为Xcode弱化了对于32位的支持，编译ffmepg时会报错：

	```
./libavutil/arm/asm.S:50:9: error: unknown directive
        .arch armv7-a
        ^
make: *** [libavcodec/arm/aacpsdsp_neon.o] Error 1

	```
	解决方法： 修改编译脚本 compile-ffmpeg.sh 中删除 对 iOS8 的 armv7 的支持：`FF_ALL_ARCHS_IOS8_SDK=“arm64 i386 x86_64”`，重新编译。
2. 打包framework时报错
	编译IJKMediaFramework时Xcode报错，因为ffmpeg已经删除了 armv7 架构，所以这里会找不到 armv7 下的配置文件。报错：
	
	 ```'armv7/avconfig.h' file not found```
	 ```'armv7/config.h' file not found```
	 
	 Xcode中找不到这两个文件，也不清楚具体路径，全盘搜索有多个avconfig.h 和 config.h 文件。
	 
	 具体路径是：
	 * config.h: `~/ijkplayer/ios/build/universal/include/libffmpeg/config.h`
	 * avconfig.h: `~/ijkplayer/ios/build/universal/include/libavutil/avconfig.h`
3. 
