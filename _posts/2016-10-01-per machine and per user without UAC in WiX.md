This post details how to build an installer that has a radio button to allow the user to either continue with a per-machine or per-user installation and then trigger User Account Control settings (admin rights) for ONLY per-machine installations.



#### Package Attributes

The package element has the attributes *‘nstallPrivileges* and *InstallScope* which can be used to set the elevation level of the installer as well as the install scope of the installation. However both of these are depreciated and produce rather confusing results, especially when used in conjunction with other approaches.

The only viable approach is using the *WixAppFolder* property . Unfortunately this isn’t implemented properly in the default dialog sets, and results in the installer always asking for admin rights. Props to Alexander Egorov for (almost) figuring out a solution, which with some changes, works exactly as desired.

![wix](http://uvinw.github.io/assets/images/2016-10-01-wix.png)

Without going into too much detail, the default Wix dialog set that gives the user a choice for separate per user and per machine installations (based on a radio button) is the WixUI_Advanced dialog. First, get this configured for the setup project. It’s as simple as referencing WixUIExtension.dll and adding a new UIRef to WixUI_Advanced in Product.wxs. After this is done, and the  advance interface works as expected, some modifications need to correct the UAC issue.

Go the WiX source code found here and browse to the *wixlib* folder. Find *WixUI_Advanced.wxs* and save this file into the VS project. The file name given does not matter, but let’s keep it the same for this post. Do note that the file name and tag id’s of a WiX source file need not be the same. For example, you can have a file named *WixUI_Advanced.wxs* in your project, and have a UI tag inside it called *WixUI_MySetup*.

1. Since Wix already has a UI reference called WixUI_Advanced, rename the UI reference in the file we just added to *WixUI_MySetup*. Now there is no confusion in which UI we’ll be using.
2. Now add a reference to the new UI file in your main WiX source file (*Product.wxs*), somewhere under the Product tag.

```
<UIRef Id=”WixUI_MySetup” />
```



#### Custom Actions

Custom Actions can be used to carry out property changes and trigger other actions. *WixUI_Advanced* already contains a bunch of these. The order and when these actions are triggered need to be changed. Since the contents of *WixUI_Advanced* can’t be changed, this is where *WixUI_MySetup* will be used.

Custom Action names used in a WiX project must be unique (across all files). Therefore the 4 Custom Actions currently found on *WixUI_MySetup* must be given new names to prevent them from colliding with the original names in *WixUI_Advanced*. Find the following **CustomAction** tags and rename them accordingly,

- *WixSetDefaultPerUserFolder* to *MyWixSetDefaultPerUserFolder*
- *WixSetDefaultPerMachineFolder* to *MyWixSetDefaultPerMachineFolder*
- *WixSetPerUserFolder* to *MyWixSetPerUserFolder*
- *WixSetPerMachineFolder* to *MyWixSetPerMachineFolder*



#### Sequences

Sequences in WiX define which actions are triggered and in which order, during events. The *InstallUISequence* defines which dialog sets will be shown to the user and in what order from the moment the setup is run. The *InstallExecuteSequence* defines the actions that will be carried out when the product is actually being installed.

4. Replace the existing sequences (in *WixUI_MySetup*) with the following code.

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

5. Add the following Publish tags.

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



#### Setting the install directory

6. Finally add the following code to *WixUI_MySetup*. This triggers a variable change depending on the (radio button) choice made by the user in the *InstallScope* dialog and will set the default install location based on it.

```
<Publish
Dialog="InstallScopeDlg" Control="Next" Event="DoAction" Value="MyWixSetDefaultPerMachineFolder" Order="3">WixAppFolder = "WixPerMachineFolder"</Publish>
 
<Publish Dialog="InstallScopeDlg" Control="Next" Event="DoAction" Value="MyWixSetDefaultPerUserFolder" Order="3">WixAppFolder = "WixPerUserFolder" </Publish>
```



#### Clean up the default template

7. The default template has an attribute called *InstallScope* in the package tag, which needs to be removed.

Chances are, the directory structure listed in the main WiX setup file (*Product.wxs*) is as follows:

```
<Directory Id="TARGETDIR" Name="SourceDir">
<Directory Id="ProgramFilesFolder">
<Directory Id="APPLICATIONFOLDER" Name="AwesomeApp" />
...
```

8. Change the second line’s id attribute from *ProgramFilesFolder* to *LocalAppData*. This makes sure the default state of the installer is to install to the user’s local app data folder, and this does not trigger a UAC dialog.

9. As a final step you will be asked to give a hard coded GUID to your components. Go to Tools > Create GUID. Copy and paste a new GUID from here into the GUID attribute of each component.

Build the project. This should end up producing an MSI that only triggers UAC when doing a per-machine installation, and per-user installations must work fine without admin rights. Whew.