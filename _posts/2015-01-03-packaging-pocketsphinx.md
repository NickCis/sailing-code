---
layout:     post
title:      Packaging PocketSphinx
date:       2015-01-03 12:00:00
summary:    An example of packaging using CMUS Pocket Sphinx
tags: sailfishos terminal sb2 rpm packaging example pocketsphinx
---

## Packaging PocketSphinx

First of all, the Mersdk must be set up and running.

For having pocketsphinx running, we'll build 2 packages:
* `sphinxbase`: with a `-devel` subpackage
* `pocketsphinx`: with a `-devel` subpackage


1. Connect to the mersdk:

    [nickcis@myhost ~]$ ssh -p 2222 -i ~/SailfishOS/vmshare/ssh/private_keys/engine/mersdk mersdk@localhost

2. Install build dependecies.

The build used are `autotools` (libtools package provides them)

    [mersdk@SailfishSDK ~]$ sb2 -t SailfishOS-armv7hl -m sdk-install -R zypper search libtool
    Loading repository data...
    Reading installed packages...
    
    S | Name                   | Summary                                                                  | Type
    --+------------------------+--------------------------------------------------------------------------+-----------
      | libtool                | The GNU Portable Library Tool                                            | package
      | libtool                | The GNU Portable Library Tool                                            | srcpackage
      | libtool-debugsource    | Debug sources for package libtool                                        | package
    i | libtool-ltdl           | Runtime libraries for GNU Libtool Dynamic Module Loader                  | package
      | libtool-ltdl-debuginfo | Debug information for package libtool-ltdl                               | package
      | libtool-ltdl-devel     | Tools needed for development using the GNU Libtool Dynamic Module Loader | package
    [mersdk@SailfishSDK ~]$ sb2 -t SailfishOS-armv7hl -m sdk-install -R zypper search libtool
    Loading repository data...
    Reading installed packages...
    
    S | Name                   | Summary                                                                  | Type
    --+------------------------+--------------------------------------------------------------------------+-----------
      | libtool                | The GNU Portable Library Tool                                            | package
      | libtool                | The GNU Portable Library Tool                                            | srcpackage
      | libtool-debugsource    | Debug sources for package libtool                                        | package
    i | libtool-ltdl           | Runtime libraries for GNU Libtool Dynamic Module Loader                  | package
      | libtool-ltdl-debuginfo | Debug information for package libtool-ltdl                               | package
      | libtool-ltdl-devel     | Tools needed for development using the GNU Libtool Dynamic Module Loader | package
    [mersdk@SailfishSDK ~]$ sb2 -t SailfishOS-armv7hl -m sdk-install -R zypper install libtool
    Loading repository data...
    Reading installed packages...
    Resolving package dependencies...
    
    The following NEW package is going to be installed:
      libtool
    
    1 new package to install.
    Overall download size: 528.8 KiB. After the operation, additional 2.2 MiB will be used.
    Continue? [y/n/?] (y): y
    Retrieving package libtool-2.4.2-1.1.1.armv7hl                                                                                                                                                                                      (1/1), 528.8 KiB (  2.2 MiB unpacked)
    Retrieving: libtool-2.4.2-1.1.1.armv7hl.rpm ..........................................................................................................................................................................................................[done (23.1 KiB/s)]
    Installing: libtool-2.4.2-1.1.1 ...................................................................................................................................................................................................................................[done]

Then install all dependcies:
* bison: `sb2 -t SailfishOS-armv7hl -m sdk-install -R zypper install bison`


3. Set up build directory (optional)

We'll create a sanbox folder, but it is something optional

    [mersdk@SailfishSDK ~]$ mkdir sandbox
    [mersdk@SailfishSDK ~]$ cd sandbox/

### Building SphinxBase

1. Download the sources and uncompress

    [mersdk@SailfishSDK sandbox]$ curl "http://ufpr.dl.sourceforge.net/project/cmusphinx/sphinxbase/0.8/sphinxbase-0.8.tar.gz"" -o sphinxbase-0.8.tar.gz
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100 3235k  100 3235k    0     0   397k      0  0:00:08  0:00:08 --:--:--  510k
    [mersdk@SailfishSDK sandbox]$ tar xf sphinxbase-0.8.tar.gz
    [mersdk@SailfishSDK sandbox]$ cd sphinxbase-0.8

2. Creating YAML file

The `.yaml` file use will be:

    Name: sphinxbase
    Summary: Common library for sphinx speech recognition.
    Version: 0.8
    Release: 1
    Group: System/Libraries
    License: BSD
    URL: http://cmusphinx.sourceforge.net/
    Description: |
        Common library for sphinx speech recognition.
    Sources:
        - http://ufpr.dl.sourceforge.net/project/cmusphinx/%{name}/%{version}/%{name}-%{version}.tar.gz
    Builder: make
    Requires:
        - bison
    PkgBR:
        - libtool
    Configure: autogen
    ConfigOptions:
        - --enable-fixed
        - --prefix=/usr/
    Files:
        - "%{_libdir}/libsphinx*"
        - "%{_bindir}/sphinx*"
    AutoSubPackages:
        - devel

**Note:** In the `Files` section, as strings start with `%`, they have to be quoted

3. Creating the `.spec` file

To create the `.spec` file we'll use `specify`:

    [mersdk@SailfishSDK sphinxbase-0.8]$ specify sphinxbase.yaml
    Q: Need to download source package: sphinxbase-0.8.tar.gz ?(Y/n) n
    Warning: NEW spec file created: sphinxbase.spec, maybe customized spec content is needed!

The generated file will be `sphinxbase.spec`:

    # 
    # Do NOT Edit the Auto-generated Part!
    # Generated by: spectacle version 0.27
    # 
     
    Name:       sphinxbase
     
    # >> macros
    # << macros
     
    Summary:    Common library for sphinx speech recognition.
    Version:    0.8
    Release:    1
    Group:      System/Libraries
    License:    BSD
    URL:        http://cmusphinx.sourceforge.net/
    Source0:    http://ufpr.dl.sourceforge.net/project/cmusphinx/%{name}/%{version}/%{name}-%{version}.tar.gz
    Source100:  sphinxbase.yaml
    Requires:   bison
    BuildRequires:  libtool
     
    %description
    Common library for sphinx speech recognition.
     
     
    %package devel
    Summary:    Development files for %{name}
    Group:      Development/Libraries
    Requires:   %{name} = %{version}-%{release}
     
    %description devel
    Development files for %{name}.
     
    %prep
    %setup -q -n %{name}-%{version}
     
    # >> setup
    # << setup
     
    %build
    # >> build pre
    # << build pre
     
    %autogen --disable-static
    %configure --disable-static \
        --enable-fixed \
        --prefix=/usr/
     
    make %{?_smp_mflags}
     
    # >> build post
    # << build post
     
    %install
    rm -rf %{buildroot}
    # >> install pre
    # << install pre
    %make_install
     
    # >> install post
    # << install post
     
    %files
    %defattr(-,root,root,-)
    %{_libdir}/libsphinx*
    %{_bindir}/sphinx*
    # >> files
    # << files
     
    %files devel
    %defattr(-,root,root,-)
    # >> files devel
    # << files devel

This file doesn't include any files for the devel package, so we edit it. We change the lasts lines:

    %files devel
    %defattr(-,root,root,-)
    # >> files devel
    # << files devel

To this ones:

    %files devel
    %defattr(-,root,root,-)
    # >> files devel
    %{_libdir}/pkgconfig/sphinxbase.pc
    %{_includedir}/%{name}
    # << files devel

**Note:** Files must be included between the `# >> files devel` tags, as they are custom editions of the spec file.


4. Build and create RPM

    [mersdk@SailfishSDK sphinxbase-0.8]$ mb2 -t SailfishOS-armv7hl -s sphinxbase.spec build

If everything goes right, you'll find a folder `RPMS` with the packages:

    [mersdk@SailfishSDK sphinxbase-0.8]$ ls RPMS/
    sphinxbase-0.8-1.armv7hl.rpm  sphinxbase-devel-0.8-1.armv7hl.rpm

5. Install the package

As `pocketsphinx depends of `sphinxbase`, it has to be installed.

    [mersdk@SailfishSDK sphinxbase-0.8]$ sb2 -t SailfishOS-armv7hl -m sdk-install -R zypper in ./RPMS/sphinxbase-0.8-1.armv7hl.rpm
    Loading repository data...
    Reading installed packages...
    Resolving package dependencies...
    
    The following NEW package is going to be installed:
      sphinxbase 
    
    1 new package to install.
    Overall download size: 296.6 KiB. After the operation, additional 876.3 KiB will be used.
    Continue? [y/n/?] (y): y
    Retrieving package sphinxbase-0.8-1.armv7hl                                                                                                                                                                                         (1/1), 296.6 KiB (876.3 KiB unpacked)
    Retrieving package sphinxbase-0.8-1.armv7hl                                                                                                                                                                                         (1/1), 296.6 KiB (876.3 KiB unpacked)
    Installing: sphinxbase-0.8-1 ......................................................................................................................................................................................................................................[done]
    [mersdk@SailfishSDK sphinxbase-0.8]$ sb2 -t SailfishOS-armv7hl -m sdk-install -R zypper in ./RPMS/sphinxbase-devel-0.8-1.armv7hl.rpm
    Loading repository data...
    Reading installed packages...
    Resolving package dependencies...
    
    The following NEW package is going to be installed:
      sphinxbase-devel 
    
    1 new package to install.
    Overall download size: 61.4 KiB. After the operation, additional 303.1 KiB will be used.
    Continue? [y/n/?] (y): y
    Retrieving package sphinxbase-devel-0.8-1.armv7hl                                                                                                                                                                                   (1/1),  61.4 KiB (303.1 KiB unpacked)
    Retrieving package sphinxbase-devel-0.8-1.armv7hl                                                                                                                                                                                   (1/1),  61.4 KiB (303.1 KiB unpacked)
    Installing: sphinxbase-devel-0.8-1 ................................................................................................................................................................................................................................[done]


### Building PocketSphinx

The process is very similar to the previous one.
1. Download the sources and uncompress

    [mersdk@SailfishSDK sandbox]$ curl ""http://ufpr.dl.sourceforge.net/project/cmusphinx/pocketsphinx/0.8/pocketsphinx-0.8.tar.gz" -o pocketsphinx-0.8.tar.gz
    [mersdk@SailfishSDK sandbox]$ tar xf pocketsphinx-0.8.tar.gz
    [mersdk@SailfishSDK sandbox]$ cd pocketsphinx-0.8

2. Creating YAML file

    Name: pocketsphinx
    Summary: recognizer library written in C.
    Version: 0.8
    Release: 1
    Group: System/Libraries
    License: BSD
    URL: http://cmusphinx.sourceforge.net/
    Description: |
        Common library for sphinx speech recognition.
    Sources:
        - http://ufpr.dl.sourceforge.net/project/cmusphinx/%{name}/%{version}/%{name}-%{version}.tar.gz

    Builder: make
    Requires:
        - sphinxbase >= %{version}
    PkgBR:
        - sphinxbase-devel
        - libtool

    Configure: autogen
    ConfigOptions:
        - --prefix=/usr/
    Files:
        - "%{_bindir}/%{name}*"
        - "%{_libdir}/lib*"
        - "%{_datadir}/%{name}"
    AutoSubPackages:
        - devel

3. Creating the `.spec` file

    [mersdk@SailfishSDK pocketsphinx-0.8]$ specify pocketsphinx.

Edit the `files devel`. The final `.spec` file will be:

    # 
    # Do NOT Edit the Auto-generated Part!
    # Generated by: spectacle version 0.27
    # 
    
    Name:       pocketsphinx
    
    # >> macros
    # << macros
    
    Summary:    recognizer library written in C.
    Version:    0.8
    Release:    1
    Group:      System/Libraries
    License:    BSD
    URL:        http://cmusphinx.sourceforge.net/
    Source0:    http://ufpr.dl.sourceforge.net/project/cmusphinx/%{name}/%{version}/%{name}-%{version}.tar.gz
    Source100:  pocketsphinx.yaml
    Requires:   sphinxbase >= %{version}
    
    %description
    Common library for sphinx speech recognition.
    
    
    %package devel
    Summary:    Development files for %{name}
    Group:      Development/Libraries
    Requires:   %{name} = %{version}-%{release}
    
    %description devel
    Development files for %{name}.
    
    %prep
    %setup -q -n %{name}-%{version}
    
    # >> setup
    # << setup
    
    %build
    # >> build pre
    # << build pre
    
    %autogen --disable-static
    %configure --disable-static \
        --prefix=/usr/
    
    make %{?_smp_mflags}
    
    # >> build post
    # << build post
    
    %install
    rm -rf %{buildroot}
    # >> install pre
    # << install pre
    %make_install
    
    # >> install post
    # << install post
    
    %files
    %defattr(-,root,root,-)
    # >> files
    %{_bindir}/%{name}*
    %{_libdir}/lib*
    %{_datadir}/%{name}
    # << files
    
    %files devel
    %defattr(-,root,root,-)
    # >> files devel
    %{_includedir}/%{name}
    %{_datadir}/man
    # << files devel


4. Build and create RPM
5. Install the package
