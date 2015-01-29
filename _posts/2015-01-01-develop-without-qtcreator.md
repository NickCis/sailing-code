---
layout:     post
title:      Develop without QtCreator
date:       2015-01-01 12:00:00
summary:    Trying to use Vim + Terminal, instead of the de facto QtCreator
categories: sailfishos qtcreator vim terminal
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

For more info `merssh`'s [source code](https://github.com/sailfish-sdk/sailfish-qtcreator/tree/next/src/tools/merssh) could be read.

To cross compile first of all we need the MerSDK Vm running.
In order to execute commands in the mersdk vm, the program `mwessh` can be used. It is located under the `bin` folder of the SailfishOS Sdk folder, using the paths of the example: `~/SailfishOS/bin/merssh`.

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

Enviromental variables:

* `MER_SSH_TARGET_NAME`: Name of the destination platform, qtcreator uses `SailfishOS-i486`
* `MER_SSH_SHARED_HOME`: Path to the shared home, by default it is your home folder
* `MER_SSH_SHARED_TARGET`: `mersdk/targets` folder inside the SailfishOS sdk folder
* `MER_SSH_SHARED_SRC`: really do not know what it is, but, qtcreator sets it to your home folder
* `MER_SSH_SDK_TOOLS`: Mer Ssh tools folder
* `MER_SSH_PROJECT_PATH`: The working project path
* `MER_SSH_DEVICE_NAME`: ?
* `MER_SSH_USERNAME`: ssh username of the mersdk
* `MER_SSH_PORT`: ssh port of the mersdk
* `MER_SSH_PRIVATE_KEY`: Path to the mersdk ssh private key

Example of variables:

    export MER_SSH_TARGET_NAME=SailfishOS-i486
    export MER_SSH_SHARED_HOME=~
    export MER_SSH_SHARED_TARGET=~/SailfishOS/mersdk/targets
    export MER_SSH_SHARED_SRC=~
    export MER_SSH_SDK_TOOLS=~/.config/SailfishBeta1/mer-sdk-tools/MerSDK/SailfishOS-i486
    export MER_SSH_PROJECT_PATH=~/.SailfishOS/examples/componentgallery
    export MER_SSH_DEVICE_NAME=
    export MER_SSH_USERNAME=mersdk
    export MER_SSH_PORT=2222
    export MER_SSH_PRIVATE_KEY=~/SailfishOS/vmshare/ssh/private_keys/engine/mersdk

Possible `<command>`:
* `qmake`
* `make`
* `deploy`
* `gcc`
* `rpm`
* `rpmbuild`


#### Building custom app

1. Qmake your project:

`~/SailfishOS/bin/merssh qmake -r -spec linux-g++`

2. Make your project:

`~/SailfishOS/bin/merssh make`

3. Deploy.

`~/SailfishOS/bin/merssh deploy --sdk`

**Note:** here you have to first start the SailfishOS emulator VM, set the `MER_SSH_DEVICE_NAME` enviromental variable to `"SailfishOS Emulator"` and then run the command

Other possible flags for `merssh deploy`:

* `--pkcon`: ?
* `--rsync`: ?
* `--sdk`: Deploy using RPM -> this option is the default when using qtcreator
* `--zypper`:

A simple script could be done in order to have all the enviromental variables setted:

    #! /usr/bin/bash
    
    export MER_SSH_TARGET_NAME=SailfishOS-i486
    export MER_SSH_SHARED_HOME=~
    export MER_SSH_SHARED_TARGET=~/SailfishOS/mersdk/targets
    export MER_SSH_SHARED_SRC=~
    export MER_SSH_SDK_TOOLS=~/.config/SailfishBeta1/mer-sdk-tools/MerSDK/SailfishOS-i486
    export MER_SSH_PROJECT_PATH=`pwd`
    export MER_SSH_DEVICE_NAME=
    export MER_SSH_USERNAME=mersdk
    export MER_SSH_PORT=2222
    export MER_SSH_PRIVATE_KEY=~/SailfishOS/vmshare/ssh/private_keys/engine/mersdk
    export PATH=$PATH:~/SailfishOS/bin
    
    /usr/bin/bash

This script will give you a bash terminal with all the enviromental variables setted and the `~/SailfishOS/bin` path exported to the `PATH`

TODO: add a simple example
