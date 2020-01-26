Build Maixpy from source code on Windows (MSYS2)
=========

The following steps shows how to build on MSYS2 on Windows 10 or Windows Server 2016

## Install dependencies

git for Windows:
* download an installation program from https://gitforwindows.org/
* execute the installation program

MSYS2:

## MSYS2 installation

* download an installation program (http://repo.msys2.org/distrib/x86_64/msys2-x86_64-20190524.exe)
* execute the installation program

```
pacman -Su
pacman -S make
pacman -S gcc
pacman -S mingw64/mingw-w64-x86_64-make
pacman -S mingw64/mingw-w64-x86_64-cmake
pacman -S mingw64/mingw-w64-x86_64-python3
pacman -S mingw64/mingw-w64-x86_64-python3-pip
pacman -S mingw64/mingw-w64-x86_64-python3-pyserial
```

## Download toolchain

Download the latest toolchain from [here](https://github.com/kendryte/kendryte-gnu-toolchain/releases), or (https://s3.cn-north-1.amazonaws.com.cn/dl.kendryte.com/documents/kendryte-toolchain-win-amd64-8.2.0-20190213.zip)
* unzip the toolchain on some folder (ex. c:/msys62/opt/)

## Get source code

* Clone by https link on Windows Command Prompt

```
mkdir c:\dev
cd c:\dev
git clone https://github.com/sipeed/MaixPy.git
cd MaixPy
```

* Then get submodules

```
git submodule update --recursive --init
```

It will regiter and clone all submodules, if you don't want to register all submodules, cause some modules is unnecessary, just execute
```
git submodule update --init
# or 
git submodule update --init path_to_submodule
# or
git submodule update --init --recursive path_to_submodule
```

## Configure project

* Open Mingw-w64 64bit prompt
* Switch path to `maixpy_k210` project director

```
cd /d/dev/MaixPy
cd projects/maixpy_k210
```

* Modify several configuration files (by using vim or other text editor)

tools\cmakeのproject.py
```
:
#gen_project_type = "Unix Makefiles"
gen_project_type = "MSYS Makefiles"
:
```

projects\maixpy_k210\CMakeLists.txt
```
:
set(CMAKE_SYSTEM_NAME Generic)
set(DCMAKE_SH="CMAKE_SH-NOTFOUND")
:
```

projects\maixpy_k210\config_default.mk
```
:
CONFIG_TOOLCHAIN_PATH="d:/cross/kendryte-toolchain/bin"
:
```

components\micropython\core\py\mkrules.mk uncomment the following lines
```
:
#ifndef DEBUG
#	$(Q)$(STRIP) $(STRIPFLAGS_EXTRA) $(PROG)
#endif
#	$(Q)$(SIZE) $$(find $(BUILD) -path "$(BUILD)/build/frozen*.o") $(PROG)
:
```

* Configure toolchain path

The default toolchain path is `/opt/kendryte-toolchain/bin`,
and default toolchain pfrefix is `riscv64-unknown-elf-`.

If you have copied toolchain to `/opt`, just use default.

Or you can customsize your toolchain path (c:/msys64/opt/kendryte-toolchain/bin) by 

```
python3 project.py --toolchain c:/msys64/opt/kendryte-toolchain/bin --toolchain-prefix riscv64-unknown-elf- config 
```

And clean config to default by command

```
python3 project.py clean_conf
```

* Configure project

Usually, just use the default configuration.

If you want to customsize project modules, execute command:

```
python3 project.py menuconfig
```

This command will display a configure panel with GUI,
then change settings and save configuration.
-> This command fails. Please disregard the error.
```
:
Scanning dependencies of target menuconfig
warning: MIC_ARRAY_ENABLE (defined at C:/dev/MaixPy/components/kendryte_sdk/Kconfig:31) has leading or trailing whitespace in its prompt
Loaded configuration 'C:/dev/MaixPy/projects/maixpy_k210/build/config/global_config.mk'
Traceback (most recent call last):
  File "C:/dev/MaixPy/tools/kconfig/genconfig.py", line 131, in <module>
    menuconfig(kconfig)
  File "C:\dev\MaixPy\tools\kconfig/Kconfiglib\menuconfig.py", line 700, in menuconfig
    print(curses.wrapper(_menuconfig))
  File "C:/msys64/mingw64/lib/python3.7\curses\__init__.py", line 73, in wrapper
    stdscr = initscr()
  File "C:/msys64/mingw64/lib/python3.7\curses\__init__.py", line 30, in initscr
    fd=_sys.__stdout__.fileno())
_curses.error: setupterm: could not find terminfo database
make[3]: *** [CMakeFiles/menuconfig.dir/build.make:57: CMakeFiles/menuconfig] Error 1
make[2]: *** [CMakeFiles/Makefile2:73: CMakeFiles/menuconfig.dir/all] Error 2
make[1]: *** [CMakeFiles/Makefile2:80: CMakeFiles/menuconfig.dir/rule] Error 2
make: *** [Makefile:118: menuconfig] Error 2
```

## Build

```
python3 project.py build
```

And clean project by:

```
python3 project.py clean
```

Clean all build files by:

```
python3 project.py distclean
```

The make system is generated from `cmake`, 
you must run

```
python3 project.py rebuild
```

to rebuild make system after you add/delete source files or edit kconfig files


## Flash (Burn) to board


For example, you have one `Maix Go` board:

```
python3 project.py -B goE -p /dev/ttyUSB1 -b 1500000 flash
```

For `Maixduino` board:

```
python3 project.py -B maixduino -p /dev/ttyUSB0 -b 1500000 -S flash
```

`-B` means board, `-p` means board serial device port, `-b` means baudrate, `-S` or `--Slow` means download at low speed but more stable mode.

the configuration saved in `.flash_conf.json` except args: `--sram`(`-s`)、`--terminal`(`-t`)、`--Slow`(`-S`)
You don't need to confiure again the next time you burn firmware, just use:
```
python3 project.py flash
```
or 
```
python3 project.py -S flash
```


More parameters help by :

```
python3 project.py --help
```


## Others

* Code conventions: TODO
* Build system: refer to [c_cpp_project_framework](https://github.com/Neutree/c_cpp_project_framework)







