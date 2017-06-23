---
title: 'A gentle intro to Wix Installer'
date: '2016-08-31 19:13:00'
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
---

There are a number of ways to create an installer for an application when it’s ready to be shipped. Some of these ways are quite specific to the application and the language used to develop it. WiX Installer is one such approach, that has been around for quite some time (still in active development) can be used for pretty much anything on the Windows platform. What's great about it is that is has a boat load of features, including bootstrapping multiple installers together, running custom .NET scripts before or after certain events, and the ability to create repair/uninstall setups. WiX always outputs an MSI (.msi), so sys-admins prefer it over exe's. 

Learning how to make a proper installer with WiX can be daunting. And the *here and there* documentation doesn't make it easy.

Here's a quick crash course to get going.



### getting started



WiX is a toolset. Download it and install it from their official website. 

There are two main ways to work with WiX.



1 - Using the command line

This is the quicker approach and can be used to build an installer in a matter of minutes. Just install WiX Toolset, write your setup file in xml format (this will be a .wxs file) and use the WiX tools to compile and link your files. 

I encountered one such installer at work, which was an old legacy installer that was passed down developers over the years. Every time the install logic needed to be changed, an unfortunate dev had to manually change the files referenced in the single (yes, single) .wxs file and make any changes to the logic. I followed suite the first time I had to work with WiX, because I didn't know better. The next time however, I decided to rewrite a new installer using the second approach outlined below. 



2 - Using Visual Studio

Using VS is most certainly the recommended way for a number of reasons. It’s easier to organize the setup files and imports, and will produce a faster workflow as it’s possible to quickly compile and find any immediate errors in the files. If the application isn’t a school project that’s going to be deleted next week, this is the way to go. So first,

- Install Visual Studio
- Install WiX Toolset (must be done after installing VS if you want the build tools to be automatically installed)
- Install the WiX Toolset Build Tools for your version of VS
- In VS, create a new project like so:  Create Project > Windows Installer XML > Setup Project




### structure of a project



What the hell are these file extensions? Let's break them down.

- .wxs files

Setup (configuration) files that WiX uses are in XML format with the file extension .wxs. These files will store what components (files and folders) are included in your installer, where they’ll be installed, their folder structure as well as the interface shown by the installer. There is a naming convention where the main setup file is named Product.wxs. Place the 'components' in here, along with your application name, version, etc.

- .wixproj file  

This file contains some meta data about your VS installer project. You can use this file to build your installer from the command line using MSBuild (used for CI, building without VS, needs .NET framework installed).



### creating a simple installer



1. First create a new installer project in VS. This results in a sample Product.wxs file.

2. Add a company name to the ‘Manufacturer’ attribute in the Product tag. The project can't build without this.

3. Add a new component to it so that the installer will actually install something. A single component usually corresponds to a single file. 

   To do this, first go into the project directory and create a new file there (eg: MyApp.txt). So this installer will installer this file to a users computer.

![wix-sample-img](http://uvinw.github.io/assets/images/2016-08-31-wix-sample.png)

Now lets add this file as a component to your installer project so that it will be copied over to the users system during the installation process. To do this, simply add a new component inside the already existing ComponentGroup named ‘ProductComponents’.

```
<Component Id='MainFile' Guid='*' Directory="INSTALLFOLDER">
    <File Id=’MyApp.txt' Name=' MyApp.txt' DiskId='1' Source='MyApp.txt' KeyPath='yes'/>
</Component>
```

Now you're ready to build the project. In a few seconds, the installer will be ready. Now go ahead and run the setup.

![wix-done-img](http://uvinw.github.io/assets/images/2016-08-31-wix-done.png)

Note that there are no install configurations available to the user. It installs to the default location set in the code. Also note that the cab file (cab1.cab) is where data required by the installer is stored, and the setup cannot function without it. These files were needed back in the day when installers had to be separated so that parts could be shipped in different CD’s or downloaded part by part. To fix this, find the MediaTemplate tag and add an EmbedCab attribute to it.

```
<MediaTemplate EmbedCab="yes" />
```

And the simple installer is done.