# pulseaudio-modules-bt

this repo is a fork of pulseaudio bluetooth modules

and adds LDAC, APTX, APTX-HD, AAC support

#### Added Codecs
|Codec|Encoding(source role)|Decoding(sink role)|Sample format(s)|Sample frequnecies|
|:---:|:---:|:---:|:---:|:---:|
|AAC |✔ |✔ |s16|8, 11.025, 12,16, 22.05, 24, 32, 44.1, 48, 64, 88.2, 96 khz|
|APTX | ✔| ✔ |s16|16, 32, 44.1, 48 khz|
|APTX HD| ✔| ✔ |s24||
|LDAC |✔ |✘|s16,s24,s32,f32|44.1, 48, 88.2, 96 khz|

APTX/APTX_HD sample format fixed to s32 in PA.
(ffmpeg do the sample format transformation)

## Usage
### Packages

[wiki/Packages](https://github.com/EHfive/pulseaudio-modules-bt/wiki/Packages)

also check issue#3

**Configure modules**

See bottom.

### General Installation

**Make Dependencies**

* pulseaudio,libpulse>=11.59.1
* bluez-libs/libbluetooth~=5.0
* libdbus
* ffmpeg(libavcodec>=58, libavutil>=56) >= 4.0
* libsbc
* libfdk-aac>=0.1.5
* libtool
* cmake
* pkg-config

**Runtime Dependencies**

* pulseaudio
* bluez
* dbus
* sbc
* libfdk-aac
* [Optional] ffmpeg(libavcodec.so, libavutil.so) --- APTX, APTX-HD support
* [Optional] [ldacBT](https://github.com/EHfive/ldacBT)/libldac (libldacBT_enc.so, libldacBT_abr.so)   --- LDAC encoding support, LDAC ABR support

Note: CMakeLists.txt check if [ldacBT](https://github.com/EHfive/ldacBT) installed; If not, it will build libldac and installing libldac to PA modules dir.
See cmake option `FORCE_BUILD_LDAC` or `FORCE_NOT_BUILD_LDAC` .

#### Build

**backup original pulseaudio bt modules**

```bash
MODDIR=`pkg-config --variable=modlibexecdir libpulse`

sudo find $MODDIR -regex ".*\(bluez5\|bluetooth\).*\.so" -exec cp {} {}.bak \;
```

**pull sources**
```bash
git clone https://github.com/EHfive/pulseaudio-modules-bt.git
cd pulseaudio-modules-bt
git submodule update --init
```

**install**

A. build for PulseAudio releases (e.g., v12.0, v12.2, etc.)
```bash
git -C pa/ checkout v`pkg-config libpulse --modversion|sed 's/[^0-9.]*\([0-9.]*\).*/\1/'`

mkdir build && cd build
cmake ..
make
sudo make install
```

B. or build for PulseAudio git master
```bash
mkdir build && cd build
cmake -DFORCE_LARGEST_PA_VERSION=ON ..
make
sudo make install
```
#### Load Modules

```bash
pulseaudio -k

# if pulseaudio not restart automatically, run
pulseaudio --start
```

### Connect device

Connect your bluetooth device and switch audio profile to 'A2DP Sink';

If there is only profile 'HSP/HFP' and 'off', disconnect and reconnect your device.

The issue has been fixed in latest bluez git master, see https://github.com/EHfive/pulseaudio-modules-bt/issues/14#issuecomment-462039332.

As an alternative, you can fix it with this [udev script](https://gist.github.com/EHfive/c4f1218a75f95b076f0387403246de78).

Run `pactl list | grep a2dp_codec` to see which codec device are using.

#### Module Aruguments

**module-bluez5-discover arg:a2dp_config**

Encoders configurations

|Key| Value|Desc |Default|
|---|---|---|---|
|ldac_eqmid|hq|LDAC High Quality|auto|
||sq|LDAC Standard Quality|
||mq|LDAC Mobile use Quality|
||auto /abr|LDAC Adaptive Bit Rate|
|ldac_fmt|s16|16-bit signed (little endian)|auto|
||s24|24-bit signed|
||s32|32-bit signed|
||f32|32-bit float|
||auto|Ref default-sample-format|
|aac_bitrate_mode|\[1, 5\]|Variable Bitrate (VBR)|5|
||0|Constant Bitrate (CBR)|
|aac_afterburner (which was "aac_after_buffer" before [359ab0](https://github.com/EHfive/pulseaudio-modules-bt/commit/359ab056e002e53978a1e0b53714d5f2e799c30f)|<on/off>|Enable/Disable AAC encoder afterburner feature|off|
|aac_fmt|s16|16-bit signed (little endian)|auto|
||s32|32-bit signed|
||auto|Ref default-sample-format|

#### Configure

edit `/etc/pulse/default.pa`

append arguments to 'load-module module-bluetooth-discover'

(module-bluetooth-discover pass all arguments to module-bluez5-discover)

    # LDAC Standard Quality
    load-module module-bluetooth-discover a2dp_config="ldac_eqmid=sq"

    # LDAC High Quality; Force LDAC/PA PCM sample format as Float32LE
    #load-module module-bluetooth-discover a2dp_config="ldac_eqmid=hq ldac_fmt=f32"


equivalent to commands below if you do not use 'module-bluetooth-discover'

    load-module module-bluez5-discover a2dp_config="ldac_eqmid=sq"

    #load-module module-bluez5-discover a2dp_config="ldac_eqmid=hq ldac_fmt=f32"

#### Others

see [Wiki](https://github.com/EHfive/pulseaudio-modules-bt/wiki)

## TODO

~~add ldac abr (Adaptive Bit Rate) supprot~~

~~add APTX , APTX HD Codec support using ffmpeg~~

~~add AAC support using Fraunhofer FDK AAC codec library~~

~~add codec switching support using latest blueZ's experimental feature~~