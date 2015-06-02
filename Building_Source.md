

## Building Java Code ##
To build the java code you only need Eclipse and Android SDK.

## Building Native libraries ##
IMSDroid contains only one native library (**tinyWRAP.so**) written in C++ as a wrapper for [doubango](http://www.doubango.org/) project. The JNI files are generated using [SWIG](http://www.swig.org/).<br />
To build **tinyWRAP** you have to build all [doubango](http://www.doubango.org/)'s libraries (ANSI-C).<br />
**tinyWRAP** depends on:
  * **tinySAK** (Swiss Army Knife): Utilities functions (SHA-1, MD5, HMAC, String, List, Timers, Thread, Mutex, Semaphore, ...)
  * **tinyNET**: Networking (DNS, DHCPv4/v6, STUN, TURN, ICE, ENUM, Sockets, ...)
  * **tinyHTTP**: HTTP stack (CONNECT, DELETE, GET, HEAD, OPTIONS, PATCH, POST, PUT, TRACE, ...)
  * **tinyXCAP**: XCAP stack (AUID manager, URL generator) without XML parser (See Java code for parsers)
  * **tinyIPSec**: IPSec SA manager. Useless for Android but you MUST have it
  * **tinySMS**: SMS over IP (SM-TL, SM-RL) for IMS/LTE networks
  * **tinySIGCOMP:** Signaling Compression
  * **tinySDP**: SDP protocol
  * **tinyRTP**: RTP/RTCP protocols
  * **tinyMSRP**: MSRP protocol (Chat and File Transfer)
  * **tinyMEDIA**: Media plugins manager (Audio, video, Codecs, sessions, MSRP, QoS, ...)
  * **tinyDAV**(Doubango Audio Video): Media plugins implementation
  * **tinySIP**: SIP/IMS stack

The project also depends on FFmpeg, x264, opencore-amr, libogg, libvorbis, libtheora, iLBC, Speex, ... but you don't need to rebuild these libraries as they are already part of doubango (**$(DOUBANGO\_HOME)/thirdparties/android/lib**).

### Checkout doubango ###
```
svn checkout http://doubango.googlecode.com/svn/trunk/ doubango-read-only
```

### Adjusting **root.mk** ###
  1. goto **$(DOUBANGO\_HOME)/android-projects** and open **root.mk** with your preferred text editor
  1. Change **$ANDROID\_NDK\_ROOT** variable to point to the NDK root directory (e.g. **/cygdrive/c/android-ndk**)
  1. Change **$ANDROID\_SDK\_ROOT** variable to point to the SDK root directory (e.g. **/cygdrive/c/android-sdk**). This step is not required if you don't wish to use **adb** utility.
  1. Change **$ANDROID\_PLATFORM** variable to point to your preferred platform root directory (e.g. **$(ANDROID\_NDK\_ROOT)/build/platforms/android-1.5**)
  1. Set **$ANDROID\_GCC\_VER** variable with your GCC version (e.g. **4.2.1**)
  1. Open new Console window
  1. Add the toolchain root binary directory to the system **$PATH** if not already done:
```
export PATH=$ANDROID_NDK_ROOT\build\prebuilt\$(HOST)\arm-eabi-4.2.1\bin:$PATH
```
> where **$HOST** is equal to **darwin-x86** on MAC OS X, **windows** on Windows XP/Vista/7 and **linux-x86** on Unix-like systems. We assume that **$(ANDROID\_GCC\_VER)** is equal to **4.2.1**.<br />
  1. Set your custom **$(CFLAGS)** flags to change the default behavior. Example:
```
export CFLAGS="–Os –DDEBUG_LEVEL=DEBUG_LEVEL_ERROR"
```
You can off course set any valid GCC **$(CFLAGS)** flags. <br />
Example of doubango's specific flags:
  * **-DTNET\_HAVE\_OPENSSL\_H=1**: Enable support for TLS (You MUST have **OpenSSL**)
  * **-DJB\_HISTORY\_SIZE=500**: Set audio jitter buffer size to 500 frames
  * **-DFLIP\_DECODED\_PICT=1**: Flip the decode video frames
  * ...

### Building libtinyWRAP.so without G729AB ###
  1. Go to the android-projects root directory:
```
cd $(DOUBANGO_HOME)/android-projects
```
  1. Build all projects:
```
../bindings/java/android/buildAll.sh

# For neon optimizations
#../bindings/java/android/buildAll.sh NEON=yes
```
You can add **NEON=yes** to enable neon optimization for armv7-a processors. You MUST use at least [NDK r4b](http://developer.android.com/sdk/ndk/index.html).<br />
The binaries will be generated under **$(DOUBANGO\_HOME)/android-projects/output**.
The shared libraries will be named **libtinyWRAP\_armv7-a.so** if built with **neon** optimizations. Otherwise it will be named **libtinyWRAP\_armv5te.so**. Rename the library as **libtinyWRAP.so** and copy it to **imsdroid\libs\$(ABI)** where **ABI** is equal to **armeabi** for the ARMv5te version or **armeabi-v7a** for ARMv7-a.

### Building libtinyWRAP.so with G729AB ###
Starting IMSDroid [revision 311](https://code.google.com/p/imsdroid/source/detail?r=311) and [doubango](http://www.doubango.org/) [revision 498](https://code.google.com/p/imsdroid/source/detail?r=498) we fully support G.729 annex A and B (CNG and VAD).<br />
<br />
G.729 should only be used for experimental purpose. G.729 includes patents from several companies and is licensed by Sipro Lab Telecom. <br />
Sipro Lab Telecom is the authorized Intellectual Property Licensing Administrator for G.729 technology and patent pool.<br />
In a number of countries, the use of G.729 may require a license fee and/or royalty fee.
<br />
Because of the licensing issue the application is not built with G.729.<br />
To build **libtinyWRAP.so** with G.729AB support:
  1. Go into $(DOUBANGO\_HOME) directory
```
cd $(DOUBANGO_HOME)
```
  1. Checkout the ARM optimized version of G729AB into **/cygdrive/c/tmp**
```
svn checkout http://g729.googlecode.com/svn/trunk/ /cygdrive/c/tmp/g729b
```
  1. Copy G729AB files into [doubango](http://www.doubango.org/) project
```
cp -f /cygdrive/c/tmp/g729b/* $(DOUBANGO_HOME)/g729b
```
  1. Build and install G729AB library
```
cd $(DOUBANGO_HOME)/android-projects
make PROJECT=g729b BT=static install

#For cleanup
#make PROJECT=g729b BT=static clean
```
  1. Build tinyWRAP with G29AB
```
../bindings/java/android/buildAll.sh G729=yes

# For neon optimizations
#../bindings/java/android/buildAll.sh G729=yes NEON=yes
```

You can add **NEON=yes** to enable neon optimization for armv7-a processors. You MUST use at least [NDK r4b](http://developer.android.com/sdk/ndk/index.html).<br />
The binaries will be generated under **$(DOUBANGO\_HOME)/android-projects/output**.
The shared libraries will be named **libtinyWRAP\_armv7-a.so** if built with **neon** optimizations. Otherwise it will be named **libtinyWRAP\_armv5te.so**. Rename the library as **libtinyWRAP.so** and copy it to **imsdroid\libs\$(ABI)** where **ABI** is equal to **armeabi** for the ARMv5te version or **armeabi-v7a** for ARMv7-a.


Et voilà