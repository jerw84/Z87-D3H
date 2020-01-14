# System Specs
- i5 4670k 

- Gigabyte Z87-D3H

- GTX 760 (soon to be RX 580 when my RX 5700 comes in the mail for my gaming rig)

- Opencore 0.5.3

- macOS 10.15.2

# Transitioning from Clover to Opencore (0.5.3)

If you are a long time clover user like me and are interested in switching over to Opencore, then this guide is for you as I will document the steps that I took to switch over to Opencore. The good news is that if your hackintosh is done through the vanilla method and no other changes were made on the macOS side, then you should be able to delete your clover EFI folder and replace it with an opencore EFI folder. The bad news is that Opencore is much more hands on compared to Clover. 

From the user's perspective, Opencore makes you manage 2 additional (drivers and ACPI) folders on top of your kext folder and your config.plist. 

The first and easiest one to manage is the drivers folder. The drivers folder contains files that tell Opencore how to behave. You just drag and drop the files you need into your EFI/OC/Drivers/ folder and you're good to go. Refer to install guide on which files you need.

The second one is the ACPI folder. This one is the biggest leap I had to take in switching over to Opencore as it required me to find out info about my specific motherboard, open ACPIsamples files, edit those sample files, compile and then save them into .aml files. The guide doesn't explicitly tell you where those files came from which was super confusing for me because they just magically appeared in the image following the previous: ![here](https://i.imgur.com/1Ssvqfw.png) to ![here](https://i.imgur.com/HVuyghf.png) 

Opencore also currently does not have a front end editing software like Clover configurator. Instead it uses something called ProperTree. If you're not familiar with coding in general, it might be a bit overwhelming as you will see terms like "boolean, string, data, etc" whereas in Clover configurator. But overall it's not too bad as much settings are usually just "Yes/True", and "No/False".

Finally, this may or may not apply to you but from AptioMemoryFix is now integrated into Opencore. For me this was an issue because my BIOS setting did not have an option for me to turn off CFG Lock. I ended up spending a few more hours fixing this: https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/extras/msr-lock as detailed by the guide. If I were to do it all over again, this would have been the first thing I checked for. If, after checking your BIOS, you don't have the option to turn CFG Lock off, you might have to start here first. 

# Annotations/Commentary of the guide


# Creating the USB

This section outlines the files you're going to need to create the EFI folder for Opencore. 

OpenCorePkg - this is your bootloader. In it, you'll find files you need to make your Opencore EFI folder look like this: 
![this](https://i.imgur.com/1Ssvqfw.png)
You'll also find ACPI samples (in .dsl form) which you will need to edit and compile into (.aml form)

ApplesupportPkg - additional drivers for Opencore.

mountEFI - this is the tool you need to use to mount EFI. If you're a Clover user, you might remember this function in Clover Configurator. 

ProperTree - use this to edit your config.plist from now on as Clover Configurator is no longer compatible.

MaciASL - not listed in this section but you need this program to convert the ACPI samples from .dsl to .aml
https://github.com/acidanthera/MaciASL/releases



# Setting Up Opencore's EFI environment
https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/creating-the-usb#setting-up-opencores-efi-enviroment

Format your USB drive, as instructed by the guide. 


# Base Folder Structure
https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/creating-the-usb#base-folder-structure

This section will be new to Clover users as Clover automated most of this. 

First, make your EFI folder look like the one in the picture. Fill in the kext folder with the appropriate kext as you'd normally do with Clover. Fill the drivers folder with the drivers from OpenCorePKG and AppleSupportPKG. Fill the ACPI folder with .aml files. <b>These .aml files are not something that are downloaded and will have to be compiled by you. Don't expect the sample files to work without making the appropriate changes as each system will be different. For example, SSDT-EC.aml or SSDT-EC-USBX.aml requires you to uncomment a section of the code to work. That is to say you need to remove /* and */ at the beginning and the end of a section of the code for it to work.</b> Refer to this section of the guide on how to do this: https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/extras/acpi

Different systems will require different files in the ACPI folder. For my system (Haswell), I followed this section of the guide to decide which files I will need:
https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/config.plist/haswell#haswell

So in my case, I needed SSDT-Plug.aml and SSDT-EC.aml. Check the guide on what you'll need for your system. Once you've made the appropriate changes to the sample .dsl files as per the guide, hit compile. If there is nothing wrong with the code, MaciASL should pop up an empty window. If there's a coding error, the pop up window will give you errors. Once you've hit compile and there are no errors, hit save as... and save as a .aml file. Put these files into your ACPI folder.

At this point in the guide, there should be kexts in your kext folder, drivers in your drivers folder, and .aml files in your ACPI folder. Don't forget to put sample.plist from OpencorePKG and rename it to config.plist in to the OC folder as well. It's a good idea to start from scratch and not use the one from Clover just in case. Don't worry about the Shell.efi in the tools folder. They aren't required for Opencore to boot. 

Take a moment to double check and make sure that your folder looks like the picture: ![here](https://i.imgur.com/HVuyghf.png) 
This is important because the next step (using ProperTree) is going to pull information from your OC folder and populate your config.plist accordingly. Not only that, but Opencore is set up so that there is a specific order in which it must follow to load kext in. For example, Lilu must and always will be the first kext it loads.


# ProperTree
Since Opencore doesn't use Clover Configurator, you're going to have to use ProperTree now to make changes to your config.plist. Open up ProperTree and the first thing you'll want to do is use the Snapshot function (Cmd/Crtl + R and point to your OC folder). This will pull all of the information of the kexts, drivers, and ACPI .aml files you have in your OC Folder. Anytime you make a change, such as adding or deleting a file, you'll want to make sure to use this function so that ProperTree updates your config.plist. For example, after I mapped my USB ports, I added SSDT-UIAC.aml to my ACPI folder. I needed to use the Snapshot function in ProperTree so that Opencore will know to also load that file at boot. 

Once ProperTree has all of your info from your OC folder, follow the rest of the guide and decide which settings fit your system accordingly. Once you're comfortable with the changes, you should be able to mount your old EFI folder, delete the old EFI folder made by Clover and replace it with the new one made with Opencore. Again, this is assuming you did the vanilla install and made no changes on the macOS side. I highly recommend doing this on a clone/back up drive first before doing it to your main drive. 

If you're looking to do a fresh install, follow this part of the guide:

https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/creating-the-usb#making-the-macos-installer

Download the OS version you want, use the command provided by Apple to make the installer to your USB.

Once the installer finishes. Mount the EFI and then copy the OC EFI. If everything is working as intended, you should be greeted with the installer screen. 









