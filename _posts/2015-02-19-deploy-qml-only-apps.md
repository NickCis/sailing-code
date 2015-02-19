---
layout:     post
title:      Deploying a qml only app
date:       2015-02-19 00:23:00
summary:    A basic idea of how to configure the qt project in order to deploy a qml-only application.
tags: sailfishos qml deploy packaging desktop pro prf
---

Links:

* [How to deploy qtqml application without sdk | Together jolla](https://together.jolla.com/question/59041/how-to-deploy-qtqml-application-without-sdk/#post-id-60298)
* [.desktop-Files | Harbour Faq](https://harbour.jolla.com/faq#4.1.0)
* [Creating Project Files | qmake Manual](http://doc.qt.io/qt-5/qmake-project-files.html)
* [Advanced usage | qmake Manual](http://doc.qt.io/qt-5/qmake-advanced-usage.html)

## App structure

A basic qt project structure is like this:

    .
    ├── <app>.desktop
    ├── <app>.pro
    ├── <app>.png
    ├── qml
    │   ├── <app>.qml
    │   └── pages
    │       └── *.qml
    ├── rpm
    │   ├── <app>.spec
    │   └── <app>.yaml
    └── src
        └── *.cpp


Where `<app>` is the name of the application.

Default sailfish configuration forces the use of the `qml` and `rpm` folder. In addition, files `<app>.desktop`, `<app>.pro` and `<app>.png` are mandatory and cannot be moved.

## `pro` File

Project or `pro` files contain all the information required by qmake to build your application. Generally, you use a series of declarations to specify the resources in the project.

A basic file will contain the followin directives:

* `TARGET`: The name of the app, this will be the name of the binary. (Remember that if you are targeting harbour, you should follow [these naming rules](https://harbour.jolla.com/faq#Naming)
* `CONFIG`: keep reading it is discussed later.
* `SOURCES`: `*.cpp` sources [optional]
* `OTHER_FILES`: [optionnal] other files included.

The `pro` file allows more directives and configuration that exceed the need of packaging a `qml` only app.

Example:

    TARGET = test
    CONFIG += sailfishapp
    SOURCES += src/test.cpp
    
    # to disable building translations every time, comment out the
    # following CONFIG line
    CONFIG += sailfishapp_i18n
    TRANSLATIONS += translations/test-de.ts

### CONFIG `prf` in `.pro` file

`qmake` can be set up with extra configuration features that are specified in feature (`.prf`) files. These extra features often provide support for custom tools that are used during the build process. To add a feature to the build process, append the feature name (the stem of the feature filename) to the CONFIG variable.

Sailfish defines 3 posible `.prf` configurations:

* `sailfishapp`: Default option for a qt project.
* `sailfishapp_qml`: Used when no building process is needed, when deploying only qml apps
* `sailfishapp_i18n`: For translations support.

The `prf` files are located in:

* For emulator (`i486`): `~/SailfishOS/mersdk/targets/SailfishOS-i486/usr/share/qt5/mkspecs/features/`
* For device (`armv7`): `~/SailfishOS/mersdk/targets/SailfishOS-armv7hl/usr/share/qt5/mkspecs/features/`

As we are developing a `qml` only app, we'll choose `sailfishapp_qml`.
The `.prf` file for this config is the following:

    TEMPLATE = aux
    
    qml.files = qml
    qml.path = /usr/share/$${TARGET}
    
    desktop.files = $${TARGET}.desktop
    desktop.path = /usr/share/applications
    
    icon.files = $${TARGET}.png
    icon.path = /usr/share/icons/hicolor/86x86/apps
    
    INSTALLS += qml desktop icon
    
    OTHER_FILES += $$files(rpm/*)

This file defines:

* Use an `aux` template, ie, In addition, a project without a target binary.
* The existance of a `qml` folder, that will be deployed on `/usr/share/$${TARGET}`, where `$${TARGET}` is the one we defined in the `.pro` file. This folder **must** be in the root folder of the project.
* The existance of a `desktop` file, that will be deployed on `/usr/share/applications`. This file **must** be called `${{TARGET}}.desktop` and be in the root folder of the project.
* The existance of an icon file (`.png`), that will be deployed on `/usr/share/icons/hicolor/86x86/apps`. This file **must** be called `${{TARGET}}.png` and be in the root folder of the project.
* All files inside the rpm folder will be added to the `OTHER_FILES`.

## `.desktop` file

As we are shipping no binary, some tweaks have to be made in order to execute the `.qml` file. A default `.desktop` file is like this:

    [Desktop Entry]
    Type=Application
    X-Nemo-Application-Type=silica-qt5
    Name=<Name of app>
    Icon=<app>
    Exec=<app>

The meaning of the entries are:
* `Type`: Default value `Application`
* `X-Nemo-Application-Type`: [optional] it defines the type of application. For Silica QT5, use `silica-qt5`, according to the faq this will make speedups for Silica Apps. Another posible value is `no-invoker` (i don't know what it means).
* `Name`: The name of the app, it doesn't have to be unique, it's the one that will be shown.
* `Icon`: The name of the icon (without `.png`), it will be looked in `/usr/share/icons/hicolor/86x86/apps`. Generally, this will be `<app>`.
* `Exec`: The executable file.
* `X-Nemo-Single-Instance`: [optional] use `no` value if, multiple instances are allowed to run.

In order to run the `.qml` file, we'll have to use `sailfish-qml <app>` in the `Exec` entry. `sailfish-qml` is a binary provided by SailfishOS that is used to run `qml`-only apps. **It is very important** that the main `qml` file is called `<app>` and is located under the `qml` folder, `sailfish-qml` run the file `/usr/share/<app>/qml/<app>`!.

## `.yaml` file

This file is the one used to create the rpm, the structure of the file is decribed in [the packaging post]({{site.baseurl}}{% post_url 2015-01-02-packaging %}). Remember to include all files.

## Example

The QtCreator base application has been modify in order to use it as an example of how to package a `qml`-only app. [Download from here]({{site.baseurl}}/example/example-qml-only.zip).




