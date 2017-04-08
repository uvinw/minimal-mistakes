There are a couple of ways to create an installer for an application after it’s ready to be shipped. Some of these ways are quite specific to the application and the language used to develop it. WiX Installer can be used for pretty much anything on the Windows platform. Here's a quick crash course to get going.



#### Getting Started

There are two main ways to work with WiX.

1. Using the command line

This is the quicker approach and can be used to build an installer in a matter of minutes. Just install WiX Toolset, write your setup file in xml format (this will be a .wxs file) and use the WiX commands to compile and link your files.

2. Using Visual Studio

Using VS is most certainly the recommended way for a number of reasons. It’s easier to organize the setup files and imports, and will produce a faster workflow as it’s possible to quickly compile and find any immediate errors in the files. If the application isn’t a school project that’s going to be deleted next week, this is the way to go. So first,

- Install Visual Studio
- Install Wix Toolset (must be done after installing VS)
- In VS, Create Project > Windows Installer XML > Setup Project





#### Structure of a Project

- .wxs files

Setup files that Wix uses are in XML format with the file extension .wxs. These files will store what components (files and folders) are included in your installer, where they’ll be installed, their folder structure as well as the interface shown by the installer. There is a naming convention where the main setup file is named Product.wxs. Place the 'components' in here, along with your application name, version, etc.

- .wixproj file  

This file contains some meta data about your VS installer project. You can use this file to build your installer from the command line using MSBuild.



#### Creating a quick installer

1. First create a new installer project in VS. This results in a sample Product.wxs file.
2. Add a company name to the ‘Manufacturer’ attribute in the Product tag.
3. Add a new component to it so that the installer will actually install something.
   To do this, first go into the project directory and create a new file there. (Eg: MyApp.txt)

![wix-sample-img](http://uvinw.github.io/assets/images/2016-08-31-wix-sample.png)

Now lets add this file as a component to your installer project so that it will be copied over to the users system during the installation process. To do this, simply add a new component inside the already existing ComponentGroup named ‘ProductComponents’.

```
<Component Id='MainFile' Guid='*' Directory="INSTALLFOLDER">
    <File Id=’MyApp.txt' Name=' MyApp.txt' DiskId='1' Source=' MyApp.txt' KeyPath='yes'/>
</Component>
```

Now build the project. In a few seconds, the installer will be ready. Now go ahead and run the setup.

![wix-done-img](http://uvinw.github.io/assets/images/2016-08-31-wix-done.png)

Note that there is no install UI or any settings shown to the user. It installs to the default location set in the code. Also note that the cab file (cab1.cab) is where data required by the installer is stored, and the setup cannot function without it. These files were needed back in the day when installers had to be separated so that parts could be shipped in different CD’s. To fix this, find the MediaTemplate tag and add an EmbedCab attribute to it.

```
<MediaTemplate EmbedCab="yes" />
```

And we're done.