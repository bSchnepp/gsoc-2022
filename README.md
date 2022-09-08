# gsoc-2022
NetBSD GSoC Final Submission

## Final Product

The main result of this project was to enable the vc4 driver under NetBSD, and
to make as much progress as possible to enabling graphics acceleration using the driver.

### [VC4 Changes for NetBSD src](https://github.com/bSchnepp/src/tree/vc4)
### [VC4 Changes for NetBSD xsrc](https://github.com/bSchnepp/xsrc/tree/vc4)

___

## Building
Both the modified src and xsrc repositories must be cloned, with their locations
known. 
Per the instructions below, some additional files supplied here will be necessary: RPI_CONFIG for the kernel configuration, files.vc4 to describe the components for the driver, and a modified files.rpi, which binds the vc4 driver into the Raspberry Pi configuration files.

The command used to build the kernel, appropriately, was
```sh
./build.sh -U -u -O <OBJDIR> -j`nproc` -m evbarm -a earmv7hf -X <XSRC LOCATION> -x tools
./build.sh -U -u -O <OBJDIR> -j`nproc` -m evbarm -a earmv7hf -X <XSRC LOCATION> -x kernel=<RPI CONFIG LOCATION>
```
within the root directory under the copy of `src`, and with `<OBJDIR>` as a clean build location. A sample RPI_CONFIG will be supplied later in this document.
<br/>
A full release, built as 
```sh
./build.sh -U -u -O <OBJDIR> -j`nproc` -m evbarm -a earmv7hf -X <XSRC LOCATION> -x release
```
can be used as the base image for the SD card. The release image under `releasedir/evbarm-earmv7hf/armv7.img.gz` can be extracted and flashed to an SD card. 
By default, this will be a generic build of NetBSD with the appropriate DeviceTree files, but by copying `<<OBJDIR>/sys/arch/evbarm/compile/RPI_CONFIG/netbsd.img` into the root directory of the SD card's FAT partition, and changing config.txt to
replace `netbsd-GENERIC.img` with `netbsd.img` to use the newly built kernel.

## Required files
[A sample RPI_CONFIG](RPI_CONFIG) <br/>
[A copy of files.vc4, to be placed in src/sys/external/gpl2/drm2/vc4/files.vc4 needed is also here](files.vc4) <br/>
For convenience, a replacement for `sys/arch/evbarm/conf/files.rpi` [is also available here.](files.rpi) <br/>

___
 
## Functional components
All of the major components are initialized, although the DSI driver typically will not finish it's setup. The main component of the vc4 driver, along with the TXP, VEC, V3D, HVS, DPI, CRTC, and HDMI drivers all load correctly on startup. However, the DSI driver was not tested.
Appropriate wrapping for memory accesses, address space mapping, clock setup and utilization, and dependent I2C devices should all be prepared correctly, as well as utilizing existing code for power domain utilization.

___
 
## Remaining components
Some issues still appear when attempting to initialize the X server. `startx` will typically crash nearly immediately. The main cause of crashes appears to be from `drm_gem_object_put_unlocked`, which seems to be related to framebuffer setup, as it is called from `drm_gem_fb_destroy` This may be related to the CRTC or HVS components.

For future work, it would be desirable to correct these issues and allow for drawing to the display, as well as wrapping new parts of the driver into individual modules, such that it could be loaded dynamically rather than being directly injected into the kernel.


