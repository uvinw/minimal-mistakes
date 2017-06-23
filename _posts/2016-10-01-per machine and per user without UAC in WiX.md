---
title: 'Proper UAC Control (for per machine / per user) in WiX'
date: '2016-10-01 19:13:00'
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
---

UAC (User Account Control) is a warning message that pops up, preventing users from making unauthorized changes to certain parts of a Windows PC. Installing to the Program Files directory (Per-Machine installation) requires administrator privileges. Per-User installations on the other hand, require no such permissions.

![img](http://uvinw.github.io/assets/images/uac.png)

Imagine you want an installer that installs to the user's App Data folder. `C:\Users\uvinw\AppData\` This makes the app available to the current user only, and does not require administrator privileges. Making this with WiX is simple enough. Make sure no 'reserved' paths are referenced in the installer and set the install type as 'per user'.

Now let's move on to a more complicated scenario. An installer that asks the user what type of installation is desired, and only triggers the UAC warning if a per machine installation is selected. So if the user doesn't have admin rights, they should be able to install the app without a problem in a local folder.

This guide requires you to have a basic understanding of how default dialog sets work in WiX, and I'll be used the Wix_Advanced dialog set to create this installer.



### package attributes



The package element has the attributes *InstallPrivileges* and *InstallScope* which can be used to set the elevation level of the installer as well as the install scope of the installation. However both of these are depreciated and produce rather confusing results, especially when used in conjunction with other configurations.

The only viable approach is using the *WixAppFolder* property . Unfortunately this isn’t implemented properly in the default dialog sets, and results in the installer always asking for admin rights (even for per-user installations). Props to Alexander Egorov for (almost) figuring out a solution, which, with some changes, works exactly as desired.

Without going into too much detail, the default Wix dialog set that gives the user a choice for separate per user and per machine installations (based on a radio button) is the WixUI_Advanced dialog. First, get this configured for the setup project. It’s as simple as referencing WixUIExtension.dll and adding a new UIRef to WixUI_Advanced in Product.wxs. After this is done, and the  advance interface works as expected, some modifications need to correct the UAC issue.

![img](http://uvinw.github.io/assets/images/2016-10-01-wix.png)

Go the WiX source code found here and browse to the *[wixlib](https://github.com/wixtoolset/wix3/tree/develop/src/ext/UIExtension/wixlib)* folder found in *wix3/src/ext/UIExtension/*. Find *WixUI_Advanced.wxs* and save this file into your VS project. The file name given does not matter, but let’s keep it the same for this post. 

Do note that the file name and tag id’s of a WiX source file need not be the same. For example, you can have a file named *WixUI_Advanced.wxs* in your project, and have a UI tag inside it called *WixUI_MySetup*.

* Since Wix already has a UI reference called WixUI_Advanced, rename the UI reference in the file we just added to *WixUI_MySetup*. Now there is no confusion in which UI we’ll be using.
* Now add a reference to the new UI file in your main WiX source file (*Product.wxs*), somewhere under the Product tag.

```
<UIRef Id=”WixUI_MySetup” />
```



### custom actions



Custom Actions can be used to carry out property changes and trigger other actions. *WixUI_Advanced* already contains a bunch of these. The order of when these actions are triggered needs to be changed. Since the contents of *WixUI_Advanced* can’t be changed, this is where *WixUI_MySetup* will be used.

Custom Action names used in a WiX project must be unique (across all files). Therefore the 4 Custom Actions currently found on *WixUI_MySetup* must be given new names to prevent them from colliding with the original names in *WixUI_Advanced*. 

* Find the following **CustomAction** tags and rename them accordingly,
  * *WixSetDefaultPerUserFolder* to *MyWixSetDefaultPerUserFolder*
  * *WixSetDefaultPerMachineFolder* to *MyWixSetDefaultPerMachineFolder*
  * *WixSetPerUserFolder* to *MyWixSetPerUserFolder*
  * *WixSetPerMachineFolder* to *MyWixSetPerMachineFolder*




### sequences



Sequences in WiX define which actions are triggered and in which order, during events. The *InstallUISequence* defines which dialog sets will be shown to the user and in what order from the moment the setup is run. The *InstallExecuteSequence* defines the actions that will be carried out when the product is actually being installed.

* Replace the existing sequences (in *WixUI_MySetup*) with the following code.

```
<InstallExecuteSequence>
    <Custom Action="MyWixSetDefaultPerUserFolder" Before="CostFinalize" />
    <Custom Action="MyWixSetDefaultPerMachineFolder" After="MyWixSetDefaultPerUserFolder" />
    <Custom Action="MyWixSetPerUserFolder" After="MyWixSetDefaultPerMachineFolder">
        ACTION="INSTALL" AND APPLICATIONFOLDER="" AND (ALLUSERS="" OR (ALLUSERS=2 AND (NOT Privileged)))
    </Custom>
    <Custom Action="MyWixSetPerMachineFolder" After="MyWixSetPerUserFolder">
        ACTION="INSTALL" AND APPLICATIONFOLDER="" AND (ALLUSERS=1 OR (ALLUSERS=2 AND Privileged))
    </Custom>
</InstallExecuteSequence>
 
<InstallUISequence>
    <Custom Action="MyWixSetDefaultPerUserFolder" Before="CostFinalize" />
    <Custom Action="MyWixSetDefaultPerMachineFolder" After="MyWixSetDefaultPerUserFolder" />
    <Custom Action="MyWixSetPerUserFolder" After="MyWixSetDefaultPerMachineFolder">
        ACTION="INSTALL" AND APPLICATIONFOLDER="" AND (ALLUSERS="" OR (ALLUSERS=2 AND (NOT Privileged)))
    </Custom>
    <Custom Action="MyWixSetPerMachineFolder" After="MyWixSetPerUserFolder">
        ACTION="INSTALL" AND APPLICATIONFOLDER="" AND (ALLUSERS=1 OR (ALLUSERS=2 AND Privileged))
    </Custom>
</InstallUISequence>
```

WiX allows you to have multiple tags (eg: multiple *InstallUISequence* tags) in multiple places of the same or even different files. Their order doesn’t matter and these instances are all merged together before building the installer.

* Add the following Publish tags.

The publish tag can be used to save property data into existing WiX variables. In this case we’re going to push some new values into variables used by dialogs that are used by *WixUI_MySetup*.
Find the following tag:

```
<Publish Dialog="InstallScopeDlg" Control="Next" Property="ALLUSERS" Value="{}" Order="2">
WixAppFolder = "WixPerUserFolder" </Publish>
```

Change the value of the Property attribute from ALLUSERS to MSIINSTALLPERUSER, the value of the Value attribute from “{}” to “1” and the “Order” attribute from “2” to “3”.

Find the following tag:

```
<Publish Dialog="InstallScopeDlg" Control="Next" Property="ALLUSERS" Value="1" Order="3">
WixAppFolder = "WixPerMachineFolder"
</Publish>
```

Change the value of the Property attribute from ALLUSERS to MSIINSTALLPERUSER, the value of the Value attribute from “1” to “{}” and the “Order” attribute from “3” to “2”.

Note that the ALLUSERS attribute must be set to “{}” and not 0, for it to be counted as false.



### setting the install directory



* Finally add the following code to *WixUI_MySetup*. This triggers a variable change depending on the (radio button) choice made by the user in the *InstallScope* dialog and will set the default install location based on it.

```
<Publish
Dialog="InstallScopeDlg" Control="Next" Event="DoAction" Value="MyWixSetDefaultPerMachineFolder" Order="3">WixAppFolder = "WixPerMachineFolder"</Publish>
 
<Publish Dialog="InstallScopeDlg" Control="Next" Event="DoAction" Value="MyWixSetDefaultPerUserFolder" Order="3">WixAppFolder = "WixPerUserFolder" </Publish>
```



### clean up the default template



* The default template has an attribute called *InstallScope* in the package tag, which needs to be removed.

Chances are, the directory structure listed in the main WiX setup file (*Product.wxs*) is as follows:

```
<Directory Id="TARGETDIR" Name="SourceDir">
<Directory Id="ProgramFilesFolder">
<Directory Id="APPLICATIONFOLDER" Name="AwesomeApp" />
...
```

* Change the second line’s id attribute from *ProgramFilesFolder* to *LocalAppData*. This makes sure the default state of the installer is to install to the user’s local app data folder, and this does not trigger a UAC dialog.
* As a final step you will be asked to give a hard coded GUID to your components. Go to Tools > Create GUID. Copy and paste a new GUID from here into the GUID attribute of each component.

Build the project. This should end up producing an MSI that only triggers UAC when doing a per-machine installation. Whew.