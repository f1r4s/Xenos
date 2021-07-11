# Xenos Injecter New Version 

#### Redesigned GUI and some more features

##### Quote

###### - Supports x86 and x64 processes and modules
###### - Kernel-mode injection feature (driver required)
###### - Manual map of kernel drivers (driver required)
###### - Injection of pure managed images without proxy dll
###### - Windows 7 cross-session and cross-desktop injection
###### - Injection into native processes (those having only ntdll loaded)
###### - Calling custom initialization routine after injection
###### - Unlinking module after injection
###### - Injection using thread hijacking
###### - Injection of x64 images into WOW64 process
###### - Image manual mapping
###### - Injection profiles

Manual map features:
###### - Relocations, import, delayed import, bound import
###### - Static TLS and TLS callbacks
###### - Security cookie
###### - Image manifests and SxS
###### - Make module visible to GetModuleHandle, GetProcAddress, etc.
###### - Support for exceptions in private memory under DEP
###### - C++/CLI images are supported (use 'Add loader reference' in this case)

Kernel manual map features are mostly identical to user-mode with few exceptions:
###### - No C++ exception handling support for x64 images (only SEH)
###### - No static TLS
###### - No native loader compatibility
###### - Limited dependency path resolving. Only API set schema, SxS, target executable directory and system directory

Supported OS: Win7 - Win10 x64

### Additional notes:
#### Injector has 2 versions - x86 and x64. Apart from obvious features x86 version supports injection of x64 images into x64 processes; x64 injector supports injection of x86 #### and x64 images into WOW64 processes. However this is only valid for native images. If you want to inject pure managed dll - use same injector version as your target process 
#### Injection of x64 images into WOW64 process is totally unpredictable. If you want to do this I would recommend to use manual mapping with manual imports option, because  
#### native loader is more buggy than my implementation in this case (especially in windows 7).

#### Restrictions:
###### - ❌ You can't inject 32 bit image into x64 process
###### - ❌ Use x86 version to manually map 32 bit images and x86 version to map 64 bit images
###### - ❌ You can't manually map pure managed images, only native injection is supported for them
###### - ❌ May not work properly on x86 OS versions
###### - ❌ Kernel injection is only supported on x64 OSes and requires Driver Test signing mode.

##### Changelog

###### Quote

```
V2.3.2
- Win10 RS4 update support

V2.3.1
- Win10 Fall Creators update support
- STATUS_UNSUCCESSFUL codes refactored
- Bug fixes

V2.3.0
- Win10 Creators Update support
- Unified injection and manual mapping (injector -> target) : x86->x86, x64->x64, x86->x64, x64->x86
- Bug fixes, stability improvements

V2.2.2
- Bug fixes, stability improvements

V2.2.1
- Win 10 10586 driver compatibility
- Minor GUI usability fixes
- Create process: working dir changed

V2.2.0
- Command line options
- Separate x86/x64 profiles
- Pure IL exe manual mapping

V2.1.4
- VS 2015 runtime
- Win10 RTM support

V2.1.3
- Win10 build 9926 support
- Win8.1 bug fixes

V2.1.2
- Fixed BSOD under win7 and win8.1 systems
- Major kernel manual map bug fixes
- Kernel logs

V2.1.1
- Added some logging

V2.1.0
- Kernel manual map for user-mode dlls
- Process handle access rights escalation

V2.0.0
- New GUI
- Injection image list
- Auto-injection
- Injection profiles
- Injection delay timers
- Kernel injection improvements - module unlinking and init routine invocation
- Win10 tech preview support
```
![xenos_2 0 0](https://user-images.githubusercontent.com/85826349/125192180-afe27300-e270-11eb-8195-8949dab2df4a.jpg)

##### Readme

```
Process selection:
Existing - select existing process from the list
New - new process will be launched before injection
Manual launch - after pressing 'Inject' button, injector will wait for target process startup

Images:
List of images you want inject
Add - add new image to the list. Drag'n'drop is also supported
Remove - remove selected image
Clear - clear image list

Advanced options:

Injection type:
Native inject - common approach using LoadLibraryW \ LdrLoadDll in newly created or existing thread
Manual map - manual copying image data into target process memory without creating section object
Kernel(New thread) - kernel mode ZwCreateThreadEx into LdrLoadDll. Uses driver
Kernel(APC) - kernel mode APC into LdrLoadDll. Uses driver
Kernel(Manual map) - kernel manual mapping. Uses driver

Native Loader options:
Unlink module - after injection, unlink module from InLoadOrderModuleList, InMemoryOrderModuleList, InInitializationOrderModuleList, HashLinks and LdrpModuleBaseAddressIndex.
Erase PE - after injection, erase PE headers
Use existing thread - LoadLibrary and init routine will be executed in the context of random non-suspended thread.

Manual map options:
Add loader reference - Insert module record into InMemoryOrderModuleList/LdrpModuleBaseAddressIndex and HashLinks. Used to make module functions (e.g. GetModuleHandle, GetProcAddress) work with manually mapped image.
Manually resolve imports - Image import and delayed import dlls will be also manually mapped instead of being loaded using LdrLoadDll.
Wipe headers - Erase module header information after injection. Also affects manually mapped imports.
Ignore TLS - Don't process image static TLS data and call TLS callbacks.
No exception support - Don't create custom exception handlers that enable out-of-image exception support under DEP.
Conceal memory - Make image memory visible as PAGE_NO_ACESS to memory query functions

Command Line:
Process command line arguments

Init routine:
If you are injecting native (not pure IL) image, this is name of exported function that will be called after injection is done. This export is called as void ( __stdcall* )(wchar_t*) function.
If you are injecting pure managed image, this is name of public method that will be executed using ICLRRuntimeHost::ExecuteInDefaultAppDomain.

Init argument:
String that is passed into init routine

Close after injection:
Close injector after successful injection

Inject delay:
Delay before injection start

Inject interval:
Delay between each image

Menu options:

Profiles->Load - load injection profile
Profiles->Save - save current settings into profile

Tools->Eject modules - open module ejection dialog
Tools->Protect self - make injector process protected (driver required)

Command line options:
--load <profile_path> - start injector and load target profile specified by <profile_path>
--run <profile_path> - imeddiately execute profile specified by <profile_path> without GUI

Kernel injection methods require system running in Test mode.

```

#### Comon problems:
1. Access denied
Quote

```
Failed to load BlackBone driver:

{Access Denied}

A process has requested access to an object, but has not been granted those access rights.
```
If you are using account with admin rights - run program as Administrator. If you are using restricted user account - enable UAC and then run as Administrator.

2. Injection failed with error code 0xC0000225. Injector failed to resolve one or more dll dependencies. Make sure you have all required dlls and proper CRT libraries. In casof kernel manual mapping, dependencies should be placed near target process executable or in system32 (SysWOW64 for 32bit processes) folder.
