---
layout:     post
title:      Develop without QtCreator
date:       2015-01-01 12:00:00
summary:    Trying to use Vim + Terminal, instead of the de facto QtCreator
tags: sailfishos qtcreator vim terminal
---

## Setting Up Enviroment

[Sailfish Os SDK](https://sailfishos.org/) and VirtualBox must be installed (TODO: this part is ommited)

Important Links:

* [Developing with SailfishOS :: hardcodes](http://hardcodes.de/SailfishOS/Developing-with-SailfishOS.pdf)
* [sailfish-qtcreator :: github](https://github.com/sailfish-sdk/sailfish-qtcreator)
* [spectacle :: github](https://github.com/mer-tools/spectacle/)

### Starting Virtual machines

Important Paths:

* Sailfish SDK installation: `~/SailfishOS/`
* Emulator: `~/SailfishOS/emulator`
* mersdk: `~/SailfishOS/mersdk`
* bin: `~/SailfishOS/bin`

#### Loading virtual box kernel drivers:

    sudo modprobe vboxdrv

#### See if `MerSDK` and `SailfishOS Emulator` vms are listed:

    [nickcis@myhost SailfishOS]$ VBoxManage list vms
    "MerSDK" {a1bb4124-9b62-4ee8-95d7-bedb468d7389}
    "SailfishOS Emulator" {6b1e1033-4b9f-49c4-92c1-15062b9cb860}

**Note:** If VMS arent listed registered:

    VBoxManage registervm ~/SailfishOS/mersdk/MerSDK/MerSDK.vbox
    VBoxManage registervm ~/SailfishOS/emulator/SailfishOS Emulator/SailfishOS\ Emulator.vbox

#### Start MerSDK VM.

Here you need to put the correct uuid of the `MerSDK` VM, you obtain it listing the VMS as explaind in 1

    [nickcis@myhost SailfishOS]$ VBoxManage list vms
    "MerSDK" {a1bb4124-9b62-4ee8-95d7-bedb468d7389}
    "SailfishOS Emulator" {6b1e1033-4b9f-49c4-92c1-15062b9cb860}
    
    [nickcis@myhost SailfishOS]$ VBoxHeadless --startvm "MerSDK"
    Oracle VM VirtualBox Headless Interface 4.3.20_OSE
    (C) 2008-2014 Oracle Corporation
    All rights reserved.

#### Start SailfishOS Emulator.

Here you need to put the correct uuid of the `SailfishOS Emulator` VM, you obtain it listing the VMS as explaind in 1.

    [nickcis@myhost SailfishOS]$ VBoxManage list vms
    "MerSDK" {a1bb4124-9b62-4ee8-95d7-bedb468d7389}
    "SailfishOS Emulator" {6b1e1033-4b9f-49c4-92c1-15062b9cb860}
    
    [nickcis@myhost SailfishOS]$ VBoxManage startvm 6b1e1033-4b9f-49c4-92c1-15062b9cb860
    Waiting for VM "6b1e1033-4b9f-49c4-92c1-15062b9cb860" to power on...
    VM "6b1e1033-4b9f-49c4-92c1-15062b9cb860" has been successfully started.

#### Check if the vms are running.

The emulator should have display a window, but the mer sdk no. You can check if both vms are running:

    [nickcis@myhost SailfishOS]$ VBoxManage list runningvms
    "MerSDK" {a1bb4124-9b62-4ee8-95d7-bedb468d7389}
    "SailfishOS Emulator" {6b1e1033-4b9f-49c4-92c1-15062b9cb860}

### Poweroff the VMS 

To poweroff the vms correctly:

    VBoxManage controlvm <name/uuid> poweroff

Example:

    [nickcis@myhost SailfishOS]$ VBoxManage controlvm "MerSDK" poweroff
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%


### Connecting through ssh

From ["How do I login into the emulator or build engine?"](https://sailfishos.org/develop-faq.html).

#### Connect to SDK.

    ssh -p 2222 -i ~/SailfishOS/vmshare/ssh/private_keys/engine/mersdk mersdk@localhost
    ssh -p 2222 -i ~/SailfishOS/vmshare/ssh/private_keys/engine/root root@localhost

#### Connect to Emulator.

    ssh -p 2223 -i ~/SailfishOS/vmshare/ssh/private_keys/SailfishOS_Emulator/nemo nemo@localhost
    ssh -p 2223 -i ~/SailfishOS/vmshare/ssh/private_keys/SailfishOS_Emulator/root root@localhost


### Using `merssh` tool

`merssh` is the bridge between our machine and mersdk's virtual machine. It is used to run commands in the vm. The idea is to avoid log in throught ssh in order to build our application. QtCreator useses this tool to cross compile and package our project. It is located under the `bin` folder of the SailfishOS Sdk folder, using the paths of the example: `~/SailfishOS/bin/merssh`.

To run this tool, we need the MerSDK VM running. Remember that `merssh` only objective is making us easier the cross compile workflow, all commands are executed on the vm, not in the host OS.

    [nickcis@myhost speech-test]$ /home/nickcis/SailfishOS/bin/merssh
    No arguments 
    
    merssh usage: 
    merssh <command> 
    environment project parameters: 
    MER_SSH_TARGET_NAME 
    MER_SSH_SHARED_HOME 
    MER_SSH_SHARED_TARGET 
    MER_SSH_SHARED_SRC 
    MER_SSH_SDK_TOOLS 
    MER_SSH_PROJECT_PATH 
    MER_SSH_DEVICE_NAME 
    evironment connection parameters: 
    MER_SSH_USERNAME 
    MER_SSH_PORT 
    MER_SSH_PRIVATE_KEY 

`merssh` recives only one parameter `<command>`, and takes all the connection and target parameters through enviromental variables. So, in order to run it correctly, first, you'll need to set up all the enviroment.

Enviromental variables:

* `MER_SSH_TARGET_NAME`: Name of the destination platform. It is the `-t` flag passed to `sb2` or `mb2`. If we log in to the mersdk vm, the command `sb2-config -l` will list the valid targets. Qtcreator uses `SailfishOS-i486` for the emulator and `SailfishOS-armv7hl` for the phone.
* `MER_SSH_SHARED_HOME`: Path to the shared home, by default it is your home folder
* `MER_SSH_SHARED_TARGET`: `mersdk/targets` folder inside the SailfishOS sdk folder
* `MER_SSH_SHARED_SRC`: I really do not know what it is, but, qtcreator sets it to your home folder
* `MER_SSH_SDK_TOOLS`: When installing, SailfishOS sdk creates a foder under `~/.config/SailfishBeta1/`, there you'll find a tools folder for each target. I don't known if merssh really uses this env var.
* `MER_SSH_PROJECT_PATH`: The working project path
* `MER_SSH_DEVICE_NAME`: I really do not what this argument really does, but is the `-d` flag passed to `sb2` or `mb2`. QtCreator uses `SailfishOS Emulator` for deploying to the emulator and leaves it blank for the phone, i'm not sure about this last thing.
* `MER_SSH_USERNAME`: ssh username of the mersdk
* `MER_SSH_PORT`: ssh port of the mersdk
* `MER_SSH_PRIVATE_KEY`: Path to the mersdk ssh private key

**Note:** `sb2` or scratchbox is a tool used in the mersdk for cross compiling (it allows you to run commands in the crosscompiling enviroment), for more information read [the packaging post]({{site.baseurl}}{% post_url 2015-01-02-packaging %}).

Example of variables:

    export MER_SSH_TARGET_NAME=SailfishOS-i486
    export MER_SSH_SHARED_HOME=~
    export MER_SSH_SHARED_TARGET=~/SailfishOS/mersdk/targets
    export MER_SSH_SHARED_SRC=~
    export MER_SSH_SDK_TOOLS=~/.config/SailfishBeta1/mer-sdk-tools/MerSDK/$MER_SSH_TARGET_NAME
    export MER_SSH_PROJECT_PATH=~/SailfishOS/examples/componentgallery
    export MER_SSH_DEVICE_NAME=
    export MER_SSH_USERNAME=mersdk
    export MER_SSH_PORT=2222
    export MER_SSH_PRIVATE_KEY=~/SailfishOS/vmshare/ssh/private_keys/engine/mersdk

Possible `<command>`:

* [`qmake`](https://github.com/sailfish-sdk/sailfish-qtcreator/blob/next/src/tools/merssh/qmakecommand.cpp#L41): executes: `mb2 -p <project path> -t <target> qmake [additional parameters/flags]`. Runs qmake in the 'build' phase.
* [`make`](https://github.com/sailfish-sdk/sailfish-qtcreator/blob/next/src/tools/merssh/makecommand.cpp#L39): executes `mb2 -p <project path> -t <target> make [additional parameters/flags]`. Runs make in the 'build' phase.
* [`deploy`](https://github.com/sailfish-sdk/sailfish-qtcreator/blob/next/src/tools/merssh/deploycommand.cpp#L41): executes `mb2 -p <project path> -t <target> -d <device> deploy [additional parameter/flags]` command, it is used to build the rpm and install it on the emulator. Additional flags could be: `--zypper|--pkcon|--rsync|--sdk` (QtCreator uses `--sdk` by default).
* [`gcc`](https://github.com/sailfish-sdk/sailfish-qtcreator/blob/next/src/tools/merssh/gcccommand.cpp#L38): from what I understand of the code is doesn't work, one would suppose that it will execute `gcc` using `sb2`, but it doesn't.
* [`rpm`](https://github.com/sailfish-sdk/sailfish-qtcreator/blob/next/src/tools/merssh/rpmcommand.cpp#L41): executes: `mb2 -p <project path> -t <target> rpm [additional parameters/flags]`. Runs the install & rpm-creation phases
* [`generatesshkeys`](https://github.com/sailfish-sdk/sailfish-qtcreator/blob/next/src/tools/merssh/generatekeyscommand.cpp#L86): From what the name says it could be used to generate the ssh keys, but i didn't try to find out how to use it or what really does.
* `rpmbuild`: this file `~/.config/SailfishBeta1/mer-sdk-tools/MerSDK/SailfishOS-i486/rpmbuild`, especially  if you read it's last line: `exec "/home/nickcis/SailfishOS/bin/merssh" rpmbuild ${ARGUMENTS}`, makes me believe in the existance of an `rpmbuild` command. But from [`merssh source code`](https://github.com/sailfish-sdk/sailfish-qtcreator/blob/next/src/tools/merssh/main.cpp#L91) you see that it doesn't. In addition, when you run `merssh rpmbuild` it gives you a `Command not found.` error. Although, I really do not known why this file exists, it would be nice to have it. Building the rpm for distribution involves running `mb2 build`, which runs the `rpmbuild` phase (I believe the name comes from there). As this command doesn't exist, in order to package the rpm for the phone you'll have to connect through ssh and execute mb2 as explained in [the packaging post]({{site.baseurl}}{% post_url 2015-01-02-packaging %}).


`[additional parameters/flags]` are the additional arguments that we pass to `merssh`, this tool will pass them to the `<command>`.

Example:

    [user@folder ~]$ merssh qmake arg1 -flag1 foo bar

Will execute (in the virutal vm):

    [mersdk@SailfishSDK ~]$ mb2 -p $MER_SSH_PROJECT_PATH -t $MER_SSH_TARGET_NAME qmake arg1 -flag1 foo bar

For more info `merssh`'s [source code](https://github.com/sailfish-sdk/sailfish-qtcreator/tree/next/src/tools/merssh) could be read.

#### `mb2` command

This tools is a wrapper of `sb2` used by QtCreator in order to compile and build the project. You won't have to manually run this command, but, as explained previously, `merssh` run this command on the mersdk.

    usage: mb2 [-t <target>] [-s <specfile>] [-d <device>] [-p <projectdir>]
               [-f <folder>] [-i] [-P] [-x]
               build [-d] [-j <n>] [<args>] | qmake [<args>] | make [<args>] | ssh <args>
               install [<args>] | installdeps <args> | rpm [<args>] | deploy <args>
               run <args>
           mb2 --version
           mb2 --help
    
      Executes a subset of build commands in the context of an rpmbuild.
      Typically called from QtCreator to perform qmake/make phases of a project.
      Note that any other build steps in the .spec file will also be run.
    
      <specfile> will be looked for in the current rpm/ dir. If there is
      more than one it must be provided.
    
      CWD is used as a base dir for installroot/ and RPMS/ to allow for
      shadowbuilds
    
      mb2 is aware of spectacle and will update the spec file if there is
      an obvious yaml file which is newer.
    
      mb2 build [-d] [-j <n>] [<args>]
                         : runs rpmbuild for the given spec file in the
                           given sb2 target. Produces an rpm package.
                         : -d     enable debug build
                         : -j <n> use only 'n' CPUs to build
                         : can use -s -t -p -i -x
    
      mb2 qmake [<args>] : runs qmake in the 'build' phase
                           Note that this also verifies target
                           build dependencies are up to date
                         : can use -s -t -p
    
      mb2 make [<args>]  : run make in the 'build' phase
                         : can use -s -t -p
    
      mb2 deploy --zypper|--pkcon|--rsync|--sdk
                         : runs the install or rpm-creation phase and then
                           copies/installs the relevant files to the device
                         : can use -s -t -p -d -i -f -x
    
      mb2 run|ssh [<args>] : runs a command (on device if --device given);
                             intended for running gdb and a gdb server
                         : can use -s -t -p -d
    
      mb2 install [<args>] : runs the 'install' phase to install to /home/deploy/installroot
                         : can use -s -t -p
    
      mb2 installdeps [<args>] : refresh the zypper cache of the given target and
                                 if required installs build-dependencies
                         : can use -s -t -p
    
      mb2 rpm [<args>]   : runs the install & rpm-creation phases
                         : can use -s -t -p -i -x
    
      -i | --increment     : increment release number in spec file
      -t | --target        : specify the sb2 target to use
      -d | --device        : specify the device
      -p | --projectdir    : when running shadow build/deploy from another dir
      -s | --specfile      : if the specfile is not in rpm/*.spec and cannot be found using -p
      -f | --shared-folder : the folder where QtCreator shares devices.xml and ssh keys
                             this option is useful if deploy option is used outside of virtual
                             machine
      -P | --pedantic      : do extra checks
      -x | --fix-version   : use the latest tag from the current git branch as package version

### SDK Control center

QtCreator has a SailfishOS tab in its UI that lets you access to the SDK Control center in order to configure and update it, manage targets and use the rpm validator. To access to this control center, you'll need the mersdk vm running. This vm creates a webserver on the port `8080` which serves the ui to configure to the control center. So, to use it, without running QtCreator, you simply have to open [http://localhost:8080/](http://localhost:8080/) in any browser. I really do not know how to change that port.

#### Building custom app

Building a qt project involves 3 steps.

1. Qmake your project.
2. Make your project.
3. Deploy.

##### Qmake your project:

`~/SailfishOS/bin/merssh qmake -r -spec linux-g++`

##### Make your project:

`~/SailfishOS/bin/merssh make`

##### Deploy.

`~/SailfishOS/bin/merssh deploy --sdk`

**Note:** here you have to first start the SailfishOS emulator VM, set the `MER_SSH_DEVICE_NAME` enviromental variable to `"SailfishOS Emulator"` and then run the command.

Other possible flags for `merssh deploy`:

* `--pkcon`: ?
* `--rsync`: ?
* `--sdk`: Deploy using RPM (this option is the default when using qtcreator)
* `--zypper`:

#### Making building easier

A simple script could be done in order to have all the enviromental variables setted:

    #! /usr/bin/bash
    
    export MER_SSH_TARGET_NAME=SailfishOS-i486
    export MER_SSH_SHARED_HOME=~
    export MER_SSH_SHARED_TARGET=~/SailfishOS/mersdk/targets
    export MER_SSH_SHARED_SRC=~
    export MER_SSH_SDK_TOOLS=~/.config/SailfishBeta1/mer-sdk-tools/MerSDK/$MER_SSH_TARGET_NAME
    export MER_SSH_PROJECT_PATH=`pwd`
    export MER_SSH_DEVICE_NAME="SailfishOS Emulator"
    export MER_SSH_USERNAME=mersdk
    export MER_SSH_PORT=2222
    export MER_SSH_PRIVATE_KEY=~/SailfishOS/vmshare/ssh/private_keys/engine/mersdk
    export PATH=$PATH:~/SailfishOS/bin

    alias build='merssh qmake && merssh make && merssh deploy --sdk'
    alias start-sdk='VBoxHeadless -startvm "MerSDK" & '
    alias start-emulator='VBoxManage startvm "SailfishOS Emulator"'
    alias emu-run='ssh -p 2223 -i ~/SailfishOS/vmshare/ssh/private_keys/SailfishOS_Emulator/nemo nemo@localhost'
    
This script has to be sourced on the project folder and it will give you a bash terminal with all the enviromental variables setted and the `~/SailfishOS/bin` path exported to the `PATH`. In addition, it will have the `build` alias which does the build & deploy workflow automatically. Also, you'll have the `start-sdk` and `start-emulator` aliases in order to start the VMS.

In order to run the application, you'll have to executed manually through ssh, for example:

    [user@host-machine ~] ssh -p 2223 -i ~/SailfishOS/vmshare/ssh/private_keys/SailfishOS_Emulator/nemo nemo@localhost /usr/bin/componentgallery

This will execute the application on the emulator, and give you the console output. The alias `emu-run` makes simpler this task

    [user@host-machine ~] emu-run /usr/bin/componentgallery

`merssh` command allows you to replace qt creator in compiling and deployment of Qt projects. The problem comes when you are trying to build something more complicated or with a different workflow that `qmake -> make -> deploy`, you'll have to `ssh` the mersdk and use `sb2` command (For further info about this last: [the packaging post]({{site.baseurl}}{% post_url 2015-01-02-packaging %}).

**NOTE:** The script enviromental flags assume that you are targeting the emulator. If you want to use it to create the rpm for the phone, you'll have to change the `MER_SSH_TARGET_NAME` (to `SailfishOS-armv7hl`) and `MER_SSH_DEVICE_NAME` to the corresponding value (i think you could leave it blank). As no `rpmbuild` or `build` command exists, you will have to run the `deploy` command and look for the package in the project folder. I think that the `deploy` phase will fail when traying to copy the package to the target, as no phone is connected.
