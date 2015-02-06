---
layout:     post
title:      Packaging a custom app or lib
date:       2015-01-02 12:00:00
summary:    A brief description on how to package a custom application or library bare handed
tags: sailfishos terminal sb2 mb2 rpm packaging
---

In order to build an package as a rpm you'll need a `.spec` file. This file has the package depencencies (build and runtime), how the source has to be compiled and what files will be included. It also has metadata about the package. The `.spec` file can be generated from a `.yaml` one, which is easyer to understand and read.

Useful links:

* [Building SDL2 with Scratch Box 2 ie. sb2](https://together.jolla.com/question/22379/porting-sdl-20-game-to-sailfish/#post-id-22493)
* [Platform SDK and SB2, Mer Wiki](https://wiki.merproject.org/wiki/Platform_SDK_and_SB2)
* [Packaging guidelines, Mer wiki](https://wiki.merproject.org/wiki/Packaging_guidelines)
* [Spectacle](https://wiki.merproject.org/wiki/Spectacle) Guidelines to generate .yaml file to create the .spec file
* [Spec Macros](http://www.rpm.org/wiki/PackagerDocs/Macros)
* [How to build a rpm by hand](https://together.jolla.com/question/7793/nemo-qml-plugin-alarms-qt5-binary-packages-missing-in-sdk-how-to-compile-from-source/#post-id-7841)
* [Packagin Applications for distribution :: SailfishOs](https://sailfishos.org/develop-packaging-apps.html)

The spectacle (`.yaml` file) and the `.spec` file are parts of the rpm workflow. They aren't Sailfish/Mer specific. Fedora/Red Hat documentation applies.

### YAML File

`.spec` file is generated automatically from the `.yaml` one, using the command `specify`. According to the [mer wiki](https://wiki.merproject.org/wiki/Packaging_guidelines#Writing_a_package_from_scratch) `.yaml` files are not mandatory, and should not be used in packages that require heavy customization.

    [mersdk@SailfishSDK ~]$ specify --help
    Usage: specify [options] [yaml-path]
    
    Options:
      --version             show program's version number and exit
      -h, --help            show this help message and exit
      -o OUTFILE_PATH, --output=OUTFILE_PATH
                            Path of output spec file
      -s, --skip-scm        Skip to check upstream SCM when specified in YAML
      -N, --not-download    Do not try to download newer source files
      -n, --non-interactive
                            Non interactive running, to use default answers
      --new=NEWYAML         Create a new yaml from template
      --newsub=NEWSUB       Append a new sub-package to current yaml

#### Tags of YAML

[YAML spec](http://www.yaml.org/spec/)
For full description refer to this [link](https://wiki.merproject.org/wiki/Spectacle#Syntax_of_spectacle_YAML)

* `Name`: [string]
* `Summary`: [string]
* `Version`: [string]
* `Release`: [string]
* `Group`: [string] valid groups are [see mer wiki](https://wiki.merproject.org/wiki/Packaging_guidelines#Summary_Tag):
  * `System/Libraries`
  * `Aplication/Multimedia`
  * TODO: ?
* `License`: [string]
* `Sources`: [list of strings]
* `Description`:
* `Configure`: valid values are:
  * `autogen`: For building a project based con autotools
  * `configure`: (default)
  * `reconfigure`:
  * `none`:
* `ConfigOptions`: [list of strings], extra flags to be passed to the configure command
* `Builder`: [string] possible values:
  * `qtc5`: Value used by default in qtcreator
  * `make`: (default) use make as builder
  * `single-make`: ?
  * `python`: ?
  * `perl`: ?
  * `qmake`: ?
  * `none`: No builder
* `Files`: [list of strings] files to be included in the rpm
* `PkgConfigBR`: [list of string] pkg-config building requires
* `Requires`: [list of string] 

**Note:** If a string starts with `%`, you'll have to quote or double quote it.

See [sphinxbase.yaml example]({{site.baseurl}}/example/sphinxbase.yaml)

### SPEC File

The spec file could be autogeneradet by the `specify` command, because of that it's not recommenden to edit it.
Custom tweaks must be done in the correct placeholders. The text between the markers will be kept as is. 

* Private Macros:
`
    # >> macros
    # << macros
`
* Extra setup scripts in the last of `%prep`
`
    # >> setup
    # << setup
`
* Pre-Build, scripts before package building
`
    # >> build pre
    # << build pre
`
* Post-Build, scripts after package building
`
    # >> build post
    # << build post
`
* Pre-Install, scripts before package installation
`
    # >> install pre
    # << install pre
`
* Post-Install, scripts after package installation
`
    # >> install post
    # << install post
`
* Files, files list in packaged rpm
`
    # >> files [sub-package]
    # << files [sub-package]
`

**Note:** sub-package is the name of the sub-package, it is optional. For the main package, the placeholder is `>> files`. If list is simple, it should have been included in the `.yaml` file.

* Scriptlets for %check section
`
    # >> check
    # << check
`

**Note:** Only if `Check` is specified as `yes`

* Scriptlets for %pre section
`
    # >> pre
    # << pre
`

**Note:** Not generated by default, if needed they must be added

* Scriptlets for %preun section
`
    # >> preun
    # << preun
`

**Note:** Not generated by default, if needed they must be added

* Scriptlets for %post section
`
    # >> post
    # << post
`

**Note:** Not generated by default, if needed they must be added

* Scriptlets for %postun section
`
    # >> postun
    # << postun
`

**Note:** Not generated by default, if needed they must be added

See [sphinxbase.spec example]({{site.baseurl}}/example/sphinxbase.spec)

For further information: [mer wiki](https://wiki.merproject.org/wiki/Spectacle#Customizations_in_spec).

#### Variables or macros
[See this link for more info](http://www.rpm.org/wiki/PackagerDocs/Macros#MacroAnaloguesofAutoconfVariables)

* `%{name}`: Package name (as specified in the yaml)
* `%{version}`: Package version (as specified in the yaml)
* `%{_prefix}`: `/usr`
* `%{_exec_prefix}`: `%{_prefix}`
* `%{_bindir}`: `%{_exec_prefix}/bin`
* `%{_sbindir}`: `%{_exec_prefix}/sbin`
* `%{_libexecdir}`: `%{_exec_prefix}/libexec`
* `%{_datadir}`: `%{_prefix}/share`
* `%{_sysconfdir}`: `%{_prefix}/etc`
* `%{_sharedstatedir}`: `%{_prefix}/com`
* `%{_localstatedir}`: `%{_prefix}/var`
* `%{_libdir}`: `%{_exec_prefix}/lib`
* `%{_includedir}`: `%{_prefix}/include`
* `%{_oldincludedir}`: `/usr/include`
* `%{_infodir}`: `%{_prefix}/info`
* `%{_mandir}`: `%{_prefix}/man`


### Building the package

[Source](https://together.jolla.com/question/22379/porting-sdl-20-game-to-sailfish/#post-id-22493)

Make sure that the "MerSDK" Virtual Box machine is running, since the cross compilation is done inside the Virtual Box. You can start it as explained in [Develop without qt-creator]({{site.baseurl}}{% post_url 2015-01-01-develop-without-qtcreator %})

You can run the compile commands through the Virtual Box window. Or log in via ssh to the build engine to run the compile commands. Login as user mersdk.

The `sb2` command and it's helper tools `sb2-*` are the tool used to do the cross compilation. The Mer wiki has a [nice introduction](https://wiki.merproject.org/wiki/Platform_SDK_and_SB2) to `sb2`. In the Build Engine the available sb2 targets are:

    [mersdk@SailfishSDK ~]$ sb2-config -l
    SailfishOS-armv7hl
    SailfishOS-i486

The `SailfishOS-armv7hl` target is used to build binaries for the device, the `SailfishOS-i486-x86` target can be used for Sailfish OS Emulator Virtual Boxes. So for any `sb2` commands you run, use the option `-t` to tell `sb2` with which target you want to operate.

    [mersdk@SailfishSDK ~]$ sb2 -t SailfishOS-armv7hl ...

With `sb2-config -d SailfishOS-armv7hl` you can configure `SailfishOS-armv7hl` to be the default target. So you don't need to provide the `-t` option.


#### Intalling dependencies

The `SailfishOS-armv7hl` target might not have all the necesary development tools or headers installed. You can search the packages: 

    [mersdk@SailfishSDK ~]$ sb2 -t SailfishOS-armv7hl -m sdk-install -R zypper search <package name>

And then, install them:

    [mersdk@SailfishSDK ~]$ sb2 -t SailfishOS-armv7hl -m sdk-install -R zypper install

#### Compile and RPM

Now you can start compiling, use the usual commands just put `sb2 -t SailfishOS-armv7hl` always in front of it.

To create an RPM package, you need to create the `.spec` file. If possible create the `.yaml` file using the instructions here, build the `.spec` and tweek it according the build engine used. To build the RPM you can use the `mb2` tool (it needs the same `-t` option as `sb2`):

    [mersdk@SailfishSDK ~]$ mb2 -t SailfishOS-armv7hl -s path/to/my_sdl_project.spec build
