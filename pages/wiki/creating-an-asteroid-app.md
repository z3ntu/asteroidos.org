---
title: Creating an Asteroid app
layout: documentation
---


# Getting the Cross Compilation Toolchain

---

The app creation process of AsteroidOS needs a Software Development Kit generated by OpenEmbedded. With the SDK installed on your computer, you can develop and compile applications for watches.

**Note:** If you intend to build apps that will run on the emulator, those applications will need to be compiled with a different version of the SDK.  Both versions of the SDK can be installed on the same computer at the same time, since they do not interfere with each other. See [below](#building-applications-for-the-emulator) for how to obtain, build, install and use that version of the SDK.

You can either grab a prebuilt SDK from the nightlies [here](https://release.asteroidos.org/nightlies/sdk/oecore-x86_64-armv7vehf-neon-toolchain-nodistro.0.sh) and install it on your system or you can build it yourself as described below.  Be aware that building it yourself will require a lot of time, internet access and disk space (over 150GiB is not unusual).

# Building the Cross Compilation Toolchain

---

Alternatively you can also build the toolchain from source. If you’ve already got an OpenEmbedded build directory via the [Building AsteroidOS]({{rel 'wiki/building-asteroidos'}}) page, cd to that directory. Else, create one with:

## Build without containers

```
git clone https://github.com/AsteroidOS/asteroid
cd asteroid/
```

Then, build the cross compilation toolchain with:

```
source ./prepare-build.sh dory
bitbake meta-toolchain-qt5
```

## Build with containers

Assuming you already prepared a docker or podman build environment like in: [Building AsteroidOS]({{rel 'wiki/building-asteroidos'}}).

```
sudo docker rm -f asteroidos-toolchain
sudo docker run -it \
  --name asteroidos-toolchain \
  -v /etc/passwd:/etc/passwd:ro \
  -u "$(id -u):$(id -g)" \
  -v "$HOME/.gitconfig:/$HOME/.gitconfig:ro" \
  -v "$(pwd):/asteroid" asteroidos-toolchain \
  bash -c "source ./prepare-build.sh dory && bitbake meta-toolchain-qt5"
```

or

```
podman run --rm -it \
  -v "$(pwd)":/asteroid:z \
  --userns keep-id asteroidos-toolchain \
  bash -c "source ./prepare-build.sh dory && bitbake meta-toolchain-qt5"
```

# Install the SDK

---

If you downloaded a prebuilt SDK, run the downloaded script. If you followed the previous steps, this will have generated the same installation script in `tmp-glibc/deploy/sdk/`, you can run it as follows:

```
./tmp-glibc/deploy/sdk/oecore-x86_64-armv7vehf-neon-toolchain-nodistro.0.sh
```

This script will install the cross-compiler and ARM libraries in `/usr/local/oecore-x86_64` by default, along with a script that needs to be sourced before every build. If you want to build a simple project via the terminal, this can be done like that:

```
source /usr/local/oecore-x86_64/environment-setup-armv7vehf-neon-oe-linux-gnueabi
cmake -B build
cmake --build build
```

# Configure QtCreator for cross compilation

---

Before running QtCreator you must run the previously mentioned script:

```
source /usr/local/oecore-x86_64/environment-setup-armv7vehf-neon-oe-linux-gnueabi
qtcreator
```
One other environment variable needs to be set to work seamlessly with AsteroidOS:

```
export CMAKE_PROGRAM_PATH=/usr/local/oecore-x86_64/sysroots/armv7vehf-neon-oe-linux-gnueabi/usr/bin/
```

This can be done automatically by prepending `source /usr/local/oecore-x86_64/environment-setup-armv7vehf-neon-oe-linux-gnueabi` and the export command before `#!/bin/sh` in `/usr/bin/qtcreator.sh`

Now that you are in QtCreator go to `Tools` &rarr; `Options` &rarr; `Devices`

- Add a new Generic Linux Device.
- Name it "AsteroidOS Watch".
- Choose 192.168.2.15 as IP address.
- Use root as user.
- Choose Password authentication and leave the password field empty.


Under the `Kits` add a kit with the previously defined device:
- Set `Device type` to `Generic Linux Device`.
- Set the `Device` to `AsteroidOS Watch`.
- Set the sysroot to `/usr/local/oecore-x86_64/sysroots/armv7vehf-neon-oe-linux-gnueabi/`.
- In the CMake generator change the `Generator` to `Unix Makefiles`.
- Change `Qt version` to `None`.
- Change `C` compiler to `<No compiler>`.
- Change `C++` compiler to `<No compiler>`.
- Change `CMake Tool` to `System CMake at /usr/local/oecore-x86_64/usr/bin/cmake`
- Clear the `CMake Configuration` fields.

Note that if these steps are not done *in this order*, QtCreator will not let you change both the `C` compiler and `C++` compiler to `<No compiler>`.  Specifically, setting `Qt version` to `None` must be done first.

# First app

---

[Asteroid-helloworld](https://github.com/AsteroidOS/asteroid-helloworld) can act as a cool QML demo app to make your first steps into AsteroidOS development easier. You can clone it, build it, install it and then modify it to follow your needs:

```
git clone https://github.com/AsteroidOS/asteroid-helloworld
cd asteroid-helloworld/
source /usr/local/oecore-x86_64/environment-setup-armv7vehf-neon-oe-linux-gnueabi
qtcreator CMakeLists.txt
```

Try to build and deploy the app. If it wasn’t already installed, a new icon should have already appeared on asteroid-launcher.

You can start by modifying occurrences of “asteroid-helloworld” to your app’s name. Then you can change the *.desktop file which describes the icon on the apps launcher. Then modify main.qml to describe your UI. To get started with QML development you can read the [official tutorial](http://doc.qt.io/qt-5/qml-tutorial.html).

# Deploy an app from QtCreator

Open the project as described in the previous sections.

- Click on the `Projects` button on the left sidebar.
- Under the `Build & Run` section click on the `Run` configuration. This opens all run settings.
- Scroll down to the `Run` settings.

Change the following `Run` settings:
- Set the `Run configuration` to `Custom Executable (on AsteroidOS Watch)`.
- Set the `Remote executable` to `invoker`. Add the `--single-instance --type=qt5 /usr/local/bin/asteroid-helloworld` command line arguments.

Change the following `Environment` variables:
- Add `XDG_RUNTIME_DIR` and set its value to `/run/user/1000`. So that the invoker works under the root user.
- (Optional) Add `QT_WAYLAND_DISABLE_WINDOWDECORATION` with value `1`. To make the app full screen and hide the titlebar.

Your app should now be able to run from the application when you click the start button in the bottom left sidebar.

# Learn QML

The Qt documentation can be found [here](https://doc.qt.io/). (AsteroidOS uses Qt5)

[qml-asteroid](https://github.com/AsteroidOS/qml-asteroid) provides APIs and UI components specific to AsteroidOS. Example uses of qml-asteroid components can be found by searching the AsteroidOS codebase: https://github.com/search?q=org%3AAsteroidOS+StatusPage&type=code

# Quickly test a QML file on the watch

QML Tester is an on-watch app to quickly test and debug QML files. You can install it by running this command on-watch:

```
opkg install qmltester
```

Then you can edit QML files by using vim and scp

```
vim scp://ceres@192.168.2.15//path/to/file.qml
```

# Tips and tricks

---

Here is a shell script to quickly build and install an app:

```
#!/bin/sh

source /usr/local/oecore-x86_64/environment-setup-armv7vehf-neon-oe-linux-gnueabi
export CMAKE_PROGRAM_PATH=/usr/local/oecore-x86_64/sysroots/armv7vehf-neon-oe-linux-gnueabi/usr/bin/
cmake -B build
cmake --build build -j -t package

file="$(ls ./build/*.ipk | sort -V | tail -n1)"
filename="$(basename $file)"
sshpass -p "<password>" scp $file ceres@192.168.2.15:/home/ceres/
sshpass -p "<password>" ssh root@192.168.2.15 opkg install --force-reinstall --force-overwrite /home/ceres/$filename
```

You can debug your app by following along system logs with:

```
journalctl -f
```

If you want to start your app from the command line, open a shell with [SSH]({{rel 'wiki/ssh'}}), connect to ceres and use invoker:

```
invoker --type=qt5 /usr/bin/asteroid-stopwatch
```

If you want to disable screen locking for easier development you can enable the demo mode of mce as root with:

```
mcetool -D on
```

## Updating language translation files
To update language translation files, and assuming you have all source files in `src` and translation files in `i18n` (which is short for "internationalization"), you can use this command to update the files.

```
lupdate-qt5 -no-obsolete src/* i18n/*.h -ts i18n/*.ts
```

# Building applications for the emulator
The primary difference between the two versions of the SDK are that the real watches use an ARM processor, while the emulator uses a simulated x86 processor.  This allows the emulator to run much faster on computers based on the x86 architecture than if they had to emulate an ARM processor.

## Prebuilt emulator SDK
You can either download a prebuilt emulator SDK from the nightlies [here](https://release.asteroidos.org/nightlies/sdk/oecore-x86_64-core2-32-toolchain-nodistro.0.sh) and install it on your system or you can build it yourself in a process very similar to that described above for creating the watch version of the SDK.

## Building and installing the emulator SDK
The only difference between building the SDK for the emulator is that instead of using `./prepare-build.sh dory` one uses `./prepare-build qemux86`.  Installation is also largely the same except for a small difference in the command used to install.  Installing the emulator version of the SDK is done with this command:

```
./tmp-glibc/deploy/sdk/oecore-x86_64-core2-32-toolchain-nodistro.0.sh
```

## Using the emulator SDK
As the emulator installer itself describes when it is first installed,

>     Each time you wish to use the SDK in a new shell session, you need to source the environment setup script e.g.
>      $ . /usr/local/oecore-x86_64/environment-setup-core2-32-oe-linux

Integration with QtCreator is not explicitly described here, but again, the process is very similar to the process for using QtCreator with the watch version of the SDK.

# Troubleshooting

The most common problems stem from not following these directions *exactly*.  QtCreator helpfully tries to find compilers and set variables, but tends to set things up for the desktop as the target rather than AsteroidOS, so it often gets things wrong.  The first step for troubleshooting with QtCreator is to go very carefully over each of the steps listed above and verify that they all match exactly.

## Could not find a package configuration file provided by "ECM"

This is most often caused not having the environment variables set up as shown [above](#configure-qtcreator-for-cross-compilation) under the topic of configuring QtCreator.  The environment variables must all be set and then you must lauch `qtcreator` *in the same shell*.  If you're not sure you've done this, an easy way to check is to try this command from the command line:

```
echo $CC
```

This should result in a line like this:
```
arm-oe-linux-gnueabi-gcc -march=armv7ve -mfpu=neon -mfloat-abi=hard --sysroot=/usr/local/oecore-x86_64/sysroots/armv7vehf-neon-oe-linux-gnueabi
```

If instead you get an empty line or some other non-ARM compiler, you may have made an error.  One common error is to run the script directly instead of running it using `source` (or `.` on some Linux distributions).  Another common error that causes the error about not finding ECM is if, in the Kit, the system CMake is used instead of the one for the AsteroidOS SDK.

## warning: The project contains C++ source files, but the currently active kit has no C++ compiler. The code model will not be fully functional.

This is not really an error but a warning.  It's the result of having correctly chosen `<No compiler>` as per the instructions above and may safely be ignored.

## file INSTALL cannot find ... .desktop

This is probably the result of a missing `CMAKE_PROGRAM_PATH`.  As mentioned above, this must be set in order for a script that generates the desktop file to be correctly found and uses.

## Remote process crashed

One possibility is that your software has a bug, but another is that the `XDG_RUNTIME_DIR` is not set to `/run/user/1000` as mentioned above.

## "I fixed it but I get the same error!"

This most often happens when something was originally wrong with the configuration, but a CMake scan was made and a possibly faulty Makefile from an earlier attempt still exists.  To fix this, choose `Build` from the menu, and then `Rescan project`.  This will run CMake again, ignoring existing cached values and forcing the recreation of a Makefile.
