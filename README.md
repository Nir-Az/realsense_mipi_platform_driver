# D457 MIPI on Jetson AGX Xavier

## Jetson AGX Xavier board setup

Please follow the [instruction](https://docs.nvidia.com/sdk-manager/install-with-sdkm-jetson/index.html) to flash JetPack to the Jetson AGX Xavier with NVIDIA SDK Manager or other methods NVIDIA provides. Make sure the board is ready to use.

Currently Supported JetPack versions are:

- 5.0.2 (default)
- 4.6.1

## Build kernel, dtb and D457 driver

The developers can set up the source code with NVIDIA's Jetson git repositories by using the provided setup script:

```
# Using setup script, recommended for developers. If JetPack version is not given, default version will be chosen.
./setup_workspace.sh [JetPack_version]
```

Or download Jetson Linux source code tarball from [JetPack 5.0.2 BSP sources](https://developer.nvidia.com/embedded/l4t/r35_release_v1.0/sources/public_sources.tbz2), [JetPack 4.6.1 BSP sources](https://developer.nvidia.com/embedded/l4t/r32_release_v7.1/sources/t186/public_sources.tbz2).

```
# JetPack 5.0.2
mkdir l4t-gcc/5.0.2
cd ./l4t-gcc/5.0.2
wget https://developer.nvidia.com/embedded/jetson-linux/bootlin-toolchain-gcc-93 -O aarch64--glibc--stable-final.tar.gz
tar xf aarch64--glibc--stable-final.tar.gz
cd ..
wget https://developer.nvidia.com/embedded/l4t/r35_release_v1.0/sources/public_sources.tbz2
tar xjf public_sources.tbz2
cd Linux_for_Tegra/source/public
tar xjf kernel_src.tbz2

# or JetPack 4.6.1
mkdir l4t-gcc/4.6.1
cd ./l4t-gcc/4.6.1
wget http://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/aarch64-linux-gnu/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar.xz
tar xf gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar.xz --strip-components 1
cd ..
wget https://developer.nvidia.com/embedded/l4t/r32_release_v7.1/sources/t186/public_sources.tbz2
tar xjf public_sources.tbz2
cd Linux_for_Tegra/source/public
tar xjf kernel_src.tbz2
```

Apply D457 patches and build the kernel image, dtb and D457 driver.

```
# if using setup script
./apply_patches.sh [--one-cam] apply [JetPack_version]

# or, if using direct download method
# ./apply_patches_ext.sh [--one-cam] ./Linux_for_tegra/source/public [JetPack_version]

Note: The `--one-cam` option applies only for JetPack 5.0.2. By setting the option it builds DT with only camera on GMSL link A. Without it the default is to build dual camera configuration.

# build kernel, dtb and D457 driver
sudo apt install build-essential bc
./build_all.sh [--no-dbg-pkg] [JetPack_version] [JetPack_source_dir]

# remove our patches from JetPack kernel source code if using setup script
# ./apply_patches.sh reset [JetPack_version]
```

Debian packages will be generated in `images` folder.

## Install kernel and D457 driver to Jetson AGX Xavier

1. Copy the Debian package `linux-image-5.10.104-d457_5.10.104-d457-1_arm64.deb` to the Jetson AGX Xavier board and install with `sudo dpkg -i linux-image-5.10.104-d457_5.10.104-d457-1_arm64.deb`. The header, libc-dev, dbg and firmware packages are optinal.

2. Edit `/boot/extlinux/extlinux.conf` primary boot option's LINUX/FDT lines to use built kernel image and dtb file:

```
LINUX /boot/Image-5.10.104-d457
FDT /usr/lib/linux-image-5.10.104-d457/tegra194-p2888-0001-p2822-0000.dtb
```

3. Make D457 I2C module autoload at boot time: `echo "d4xx" | sudo tee /etc/modules-load.d/d4xx.conf`

After rebooting Jetson, the D457 driver should work.

**NOTE**

- Each JetPack version's kernel may be different, the user needs to change the kernel version in file names and paths accordingly, for example for JetPack 4.6.1 the version is `4.9.253-d457`.
- For JetPack 4.6.1, the dtb file is not included in the deb package. User needs to manually copy `images/4.6.1/arch/arm64/boot/dts/tegra194-p2888-0001-p2822-0000.dtb` file to board and edit `extlinux.conf` to point to it.
- It's recommended to save the original kernel image as backup boot option in `/boot/extlinux/extlinux.conf`.