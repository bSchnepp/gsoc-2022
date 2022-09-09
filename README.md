# gsoc-2022
NetBSD GSoC Final Submission

## Final Product

The main result of this project was to enable the vc4 driver under NetBSD, and
to make as much progress as possible to enabling graphics acceleration using the driver.

### [VC4 Changes for NetBSD src](https://github.com/bSchnepp/src/tree/vc4)
### [VC4 Changes for NetBSD xsrc](https://github.com/bSchnepp/xsrc/tree/vc4)

The main changes in the xsrc repository were to include necessary header files for Broadcom's VideoCore 4 GPU, as well as incorporating a patch to allow for additional platform support.
For the main src repository, most of the changes are present under `sys/external/gpl2/drm2/dist/drm/vc4` with some others in locations such as `sys/external/gpl2/drm2/vc4`, `sys/external/bsd/drm2` as well as some new DeviceTree files under `sys/arch/arm/dts/`.

## Work Done
___
 
## Timeline
For the first week, the main changes were to attempt to build each part of the vc4 driver as-is within the kernel, including the creation of some wrapper code which was missing. Most of these changes were for correcting spinlocks and other simple code.
An initial split was also done for each component of the vc4 driver, so that it can be loaded more naturally. <br/> <br/>

The second week had changes mostly consisting of correcting remaining areas where physical addresses were used under Linux structures with NetBSD equivalents, including replacing macros utilizing `writel`/`readl` with the corresponding NetBSD `bus_space` equivalent. 
Much more for each component was modified to allow for proper setup for each part as discrete components, including setting up new attach() functions to perform initialization. <br/> <br/>

The focus for the third week was to work on clocks, and prepare them for proper usage by these new drivers. Some reformatting of the previous week's work was also done for consistency. 
Functions such as `vc4_ioremap_regs` were also removed, as they are not relevant for support under NetBSD: the `fdt` code provides support for mapping in peripheral registers already. Other unnecessary portions were removed as needed. <br/> <br/>

Real hardware testing began properly for the fourth week, looking into immediate and clear issues resulting from loading the drivers. Some of these issues were resolved by enforcing a specific load order for components such as the TXP module, and ensuring
the main vc4 component loaded before all others began. Some issues with the HDMI driver and it's clocks were also found, and resolved by utilizing existing Raspberry Pi-specific code and extending it slightly to also cover the use case needed for the HDMI driver.
The userland portion of the driver was also set up, with some initial patches to get it to compile. For missing components, significant investigation into the appropriate DeviceTrees was also done. <br/> <br/>

Analysis of the DeviceTree files was the main focus for the fifth week, looking into how it differs from Linux and what changes are necessary. Likewise, some investigation into the dependencies between driver components was done, looking for creating a specific 
load order for each component which will always be safe to use. Some changes necessary for the DRM implementation for NetBSD were also integrated, which was useful in looking into problems with the HDMI driver and how different parts fit together.
In addition, some restructuring was done for how each module was wrapped, to more naturally fit with the expected model for DRM.<br/> <br/>

Work done for the sixth week included fixes for the DeviceTrees, allowing modules to load as expected, with corrections to how memory was set up for it. Some problems with the framebuffer setup were found and looked through, and various fixes made throughout.
The main component that was focused on was the vc4_hvs module, which is necessary for most of the other components of the driver. In addition, some setup warnings appear when the vec driver is initialized, which was looked over and some parts of the driver
were moved around to ensure the expected invariants for DRM code were met.<br/> <br/>

Starting the second half of the project for the seventh week, mmap_object was looked at, some warnings with driver initialization, as well as problems with the DPI driver. Some relaxations of expectations within the DPI driver were necessary to allow the driver to finish loading and initialize: the clock needed was set up and prepared later.
To implement the `mmap_object` function for the vc4 driver, it was necessary to look over the existing implementation of `drm_gem_mmap_object`. This was used as a reference to be combined with the expectations of existing driver code to finish the expected parts of the function, as well as incorporate the necessary code
for setup of memory in general. Problems with power domains were also looked into, correcting a problem which prevents the `vc4_v3d` module from loading.<br/> <br/>

Next, a significant effort was placed into setup of the DSI drivers, aiming for them to have their clocks set up correctly. As a part of this process, rewriting some of the IRQ setup routines was necessary to avoid going through an unnecessary compatibility layer in new NetBSD code.
Some of the problems with the crtc driver, mainly errors in how matching it's compatible field was handled, was also fixed. At this point, it was now clear that all modules should now be doing at least something on system startup. 
A more complex method of specifically identifying which CRTC component was being used was also implemented: the VideoCore 4 has 3 different PixelValve devices, which are used as CRTCs for the DRM driver. By parsing the identification info provided at the `attach` function, which PixelValue is which can be identified.<br/> <br/>

The ninth and tenth week saw a significant effort towards enabling the I2C device needed for the HDMI driver. This is loaded as a separate component under an existing NetBSD driver, and can thus be controlled with the appropriate functions. This seems to be the only major difference, other than sound support,
between this adaptation of the driver and the original Linux driver code. By looking into various ways that the existing code can be called, the I2C component was made functional by wrapping the expected interface with native NetBSD code. The GPIO controller
code associated with the HDMI driver was also investigated, looking for problems which could arise. The main part was to verify what I2C device was being used: BSC2 is what was expected, since this is the component which corresponds to the I2C controller for the HDMI module.
To select this correctly, careful processing of the name and the fields offered during the `attach` function were needed to find exactly which controller needed to be used. The remainder of the tenth week then had the TXP module looked into for any problems it may
have related to the original Linux code: a significant part of the DRM driver stack has not yet been implemented by NetBSD, and is itself a significant project. To wrap the necessary parts of the driver in, code was brought from the current version of Linux at the time
to implement most of `drm_writeback_connector` as a separate file loaded as part of the vc4 driver. This allows for the missing DRM functionality to be suitably implemented.<br/> <br/>

For the next week, some problems for device memory allocation were solved, mainly with `vc4_dev`. In addition, `vc4_fault` was properly implemented, as this function will be necessary for demand paging with physical memory associated with the VC4 device.
Many of the warnings and errors relaxed in previous weeks were also re-strengthened, particularly for some of the formerly not yet implemented code blocks in areas since implemented correctly. Some reading of the `vc4_plane` code led to some fixes needed
for this part, which is necessary for the framebuffer setup code. Likewise, differences between how Linux handles deferred jobs and how NetBSD achieves this were also corrected in `vc4_gem`. As Linux uses a list, but NetBSD uses queues for these jobs,
this required some deep changes in the data structures used by various components leading up to the `vc4_gem` code, as well as how the entries are processed.

The twelfth week had revolved around an incorporated patch which was needed to allow much of the provided DRM functions to be properly called. The problems are now fairly isolated to framebuffer initialization, and specifically with teardown and buildup of a new one.
Other than this, all of the component drivers are attached correctly, and other than the DSI driver, all will complete their initialization with the expectations provided. Remaining issues with the DeviceTree, work_struct queues, and lock initialization
were handled. In addition, a change was also provided to enable support for the Raspberry Pi 3A+, as it is already fairly similar to the 3B+ and all functionality expected appears to work for it as well. Lastly, the last remaining unnecessary allocation function
for the `vc4_drv.c` `attach` function was also eliminated, making sure that this driver is consistent with the structure of others.

Finally, the last week of the project saw some cleanup of no longer necessary header files and other wrappers. Other minor changes were also present, which corrected a mistake in the TXP driver referencing the wrong function, as well as removing
unnecessary included headers from each of the files as needed. The main change for this week was the final report, as well as the creation of this document. <br/> <br/>

## Functional components
All of the major components are initialized, although the DSI driver typically will not finish it's setup. The main component of the vc4 driver, along with the TXP, VEC, V3D, HVS, DPI, CRTC, and HDMI drivers all load correctly on startup. 
However, the DPI driver was not tested. Appropriate wrapping for memory accesses, address space mapping, clock setup and utilization, and dependent I2C devices should all be prepared correctly, 
as well as utilizing existing code for power domain utilization.

___
 
## Remaining components
Some issues still appear when attempting to initialize the X server. `startx` will typically crash nearly immediately. The main cause of crashes appears to be from `drm_gem_object_put_unlocked`, which seems to be related to framebuffer setup, as it is called from `drm_gem_fb_destroy` This may be related to the CRTC or HVS components. Some other issues seem to also occasionally appear rarely, but seem to be related to the same underlying cause.

For future work, it would be desirable to correct these issues and allow for drawing to the display, as well as wrapping new parts of the driver into individual modules, such that it could be loaded dynamically rather than being directly injected into the kernel.

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
