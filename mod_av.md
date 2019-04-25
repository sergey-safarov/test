# About

This page describes how to build specific FreeSwitch mod_av version and install to server.

# Preparation

First need install compilation tools
```sh
apt-get update
apt-get install git curl make automake autoconf gcc \
    libtool-bin yasm g++ pkg-config uuid-dev libjpeg-dev \
    libsqlite3-dev libcurl4-openssl-dev libpcre3-dev \
    libspeex-dev libspeexdsp-dev libldns-dev libedit-dev \
    libtiff-dev liblua5.2-dev libopus-dev libsndfile-dev \
    libavformat-dev libswscale-dev
```
Configure FreeSwitch repo
```sh
curl https://files.freeswitch.org/repo/deb/debian/freeswitch_archive_g0.pub | apt-key add -
echo "deb http://files.freeswitch.org/repo/deb/freeswitch-1.6/ jessie main" > /etc/apt/sources.list.d/freeswitch.list
apt-get update
```


# Module compilation
Need download FreeSwitch sources
```sh
git clone https://freeswitch.org/stash/scm/fs/freeswitch.git
cd freeswitch
git checkout dd0bb0e331a0e4d9dfc23b16bf84b8b0f3fbfe18
```
Compile FreeSwitch
```sh
./bootstrap.sh
./configure
make
```
Compile mod_av
```sh
make -C src/mod/applications/mod_av
strip src/mod/applications/mod_av/.libs/mod_av.so
```
Compiled module will be located at `src/mod/applications/mod_av/.libs/` directory.

# Module instalation
Need to copy `src/mod/applications/mod_av/.libs/mod_av.so` file to your server at `/tmp` directory.
Then locate folder where `FreeSwitch` modules is installed.
```sh
find / -name mod_sofia.so 2> /dev/null | xargs dirname
```
This command will output directory name like `/usr/local/freeswitch/mod`. This directory name need use in next command.
```sh
cp /tmp/mod_av.so /usr/local/freeswitch/mod
```
Then need check loaded modules at FreeSwitch startup. To do this need locate `modules.conf.xml` using command
```sh
find / -name modules.conf.xml 2> /dev/null
```
And check content of this file. Module `mod_av` must be uncommented like
```xml
    <!-- File Format Interfaces -->
    <load module="mod_av"/>
```
If file not contains `<load module="mod_av"/>`, then need add this string to file.

Then need check `FreeSwitch` codec preferences. Typically this settings located at `vars.xml`. To locate this file execute
```sh
find / -name vars.xml 2> /dev/null
```
This file contains strings with `global_codec_prefs` and `outbound_codec_prefs` vars. Like this
```xml
  <X-PRE-PROCESS cmd="set" data="global_codec_prefs=OPUS,G722,PCMU,PCMA,VP8"/>
  <X-PRE-PROCESS cmd="set" data="outbound_codec_prefs=OPUS,G722,PCMU,PCMA,VP8"/>
```
Need to add codec `H263`, `H263-1998` and `H264` to this strings. Resulted strings must be looks like
```xml
  <X-PRE-PROCESS cmd="set" data="global_codec_prefs=OPUS,G722,PCMU,PCMA,VP8,H263,H263-1998,H264"/>
  <X-PRE-PROCESS cmd="set" data="outbound_codec_prefs=OPUS,G722,PCMU,PCMA,VP8,H263,H263-1998,H264"/>
```
After this need to restart `FreeSwitch` daemon.
```sh
systemctl restart freeswitch
```
And make tests.

# Notes
If on your server is used custom config, then you may not found `vars.xml` file or not found strings with `global_codec_prefs`
and `outbound_codec_prefs` vars. In this case please save `FreeSwitch` config using command and send me resulted file `/tmp/fs_cfg.tar.gz`.
```sh
tar czf /tmp/fs_cfg.tar.gz /etc/freeswitch /usr/local/freeswitch/conf
```
I will check your server config and provide instructions how to update config.