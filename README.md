# Samsung-NP900X5T-Hackintosh
make a super light laptop woks in Mac OS X majove
Mojave 10.14.6. Bios version 2018.10.26

# Hardware Configuration

Samsung NP900X5T KW02:

- Intel i5-8250U
- 8GB RAM
- Samsung 970 evo Plus 1TB NVMe SSD (latest firmware)
- Apple Magic Mouse
- UGreen usb-c Hub(with ethernet and Hdmi)
- realtek USB wifi dongle

# Working

- Intel Graphics Accelleration(uhd620)
- Build-in Bluetooth (Must enter Win10 first to have the driver loaded, then restart to enter OSX)
- Audio
- Brightlight Control
- Sleep and Wake (Hibernatemode 0, In order to wake the laptop, Should press power button until the power indictor turn blue )
- PS/2 Keyboard
- touchpad
- HDMI port
- type c hot plug
- Card Reader

# Not Working
- Build-in wifi
- Fn +several Samsung express key

# Installation

- 1.Use the Clover EFI Mojave Installer to install Mac OS Majave(10.14.6)
- 2.Boot into the system and replace the "EFI" Folder on you System Boot Disk EFI partition
- 3.Install all the kext contained in the Kext "other" folder into your /Library/Extensions, rebuild kextcache and reboot
##  ACPI Patched: 
    DSDT, SSDT-EC(for 10.15), SSDT-PNLF, SSDT-TYPC,SSDT-UIAC, SSDT-XHC, SSDT-XOSI, SSDT-YTBT
##  UEFI Drivers are used: 
    ApfsDriverLoader, AptioInputfix, AptioMemoryFix, DataHubDxe, EmuVariableUefi, 
  	FSinject, HFSPlus, NvmExpressDxe, SMCHelper.
##  Kexts are used: 
    ACPIBatteryManager, AppleALC, AppleBacklightFixup, FakeSMC, Lilu, NoTouchID, USBInjectAll,VoodooI2C, VoodooI2CHID, 
  	VoodooPS2Controller(v2.02, v2.1 is not compatible), WhateverGreen, XHCI-unsupported.
##  additional: 
    for USB Ethernet:AX88179_178A
    for USB wifi:   RtWlanU, RtWlanU1827 (Should be put under \clover\kext\others, Clover SystemParameters: InjectKexts must be changed to Yes)

# Clover Config

- Acpi: AutoMerge, DSDT Patches(_OSI to XOSI, HECI to IMEI,GFX0 to IGPU, HDAS to HDEF,no need to rename EHC* as Chipsets post Skylake removed USB2.0 native support), SSDT PluginType checked
- Boot Args: -cdfon dart=0 agdpmod=vit9696 darkwake=0 -v
- Kernel Patches: Kernel LAPIC, KernelPM and AppleRTC enabled
- SMBIOS: MacBookPro14,1
- SystemParameters: InjectKexts Detect, InjectSystemID YES.

# DSDT Patch (only 3 static patches needed )

##  Sleep and wake

    Use "USB _PRW 0x6D (instant wake)" patches for Skylake(and later). This patch will add 
	  Method(_PRW) { Return(Package() { 0x6D, 0 }) } to relevant Devices.

##  Brightness adjustment keys

    working by modifying /EFI/Clover/ACPI/patched/DSDT.aml
    Scope (_SB.PCI0.LPCB.H_EC) {
    ...
    Method (_Q63, 0, NotSerialized)  // _Qxx: EC Query
    {
        Notify (PS2K, 0x0405) // Brightness down
    }
    Method (_Q64, 0, NotSerialized)  // _Qxx: EC Query
    {
        Notify (PS2K, 0x0406) // Brightness up
    }
    ...
    }
 
 ##  Battery Patch
    Use custom patch I create for NP900X5T
 
 ##  Hot Patches -

     1.Fake EC for Catalina compatibility: SSDT-EC
     2.Backlight Control: SSDT-PNLF (RehabMan version)
     3.TypeC hot plug: SSDT-TYPC
     4.Removing unused USB ports: SSDT-UIAC
     5.inject properties for XHCI: SSDT-XHC
     6.XOSI simulation to "Windows 10": SSDT-XOSI
     7.solve recursion issue for USB-C hotplug: SSDT-YTBT
 
#  USB

##  Actual Port Information: device"8086_9d2f"

    Port		       Type	            Description
    HS01/SS01  USB 3.0 Type A	    Rear--Left side
    HS03/SS03	 USB 3.0 Type A	    Front--Right side
    HS05	     USB 2.0	          Rear--Right Side
    HS06       Proprietary        Bluetooth
    HS07	     Proprietary	      Webcam
    HS08	     Proprietary	      Fingerprint
    HS09       USB Type C         Front--Left side
    SS04       USB 3.0            Card Reader

    Use above information to make specific Hotpatch SSDT-UIAC.aml(based on SSDT-UIAC-ALL).     
##  Audio

    Realtek ALC256: Use AppleALC.kext, Clover Audio injection =56
##  HDMI

    USE the latest Hacktool to creat a patch,the below section should be take care:      
  
    framebuffer-con1-busid                        05000000(the only work-out id) 
    framebuffer-con1-enable                       01000000
    framebuffer-con1-type                         00080000(used as a HDMI identifier),


    As the MacbookPro 14,1 do not have HDMI port actually, the check of the board-id should be ignored by 
    using the WhateverGreen boot-arg agdpmod=vit9696.
  
##  EDID
    For some laptops, build-in screen will turn off if boot up with external display.this could be fixed by change the EDID ProductID to 0x9C7C
    Clover configuration:
    Graphics EDID:
               Inject    Yes
               ProductID 0x9C7C. (String)
    Devices  Properties:
           PciRoot(0x0)/Pci(0x2,0x0)
           AAPL00,override-no-connect    <>  (Data)
           
    Kindly note that the kext of WhateverGreen, Lilu and AppleALC should be updated to the latest version. 
#  Credit:

- Special thanks to RehabMan for his splendid work and comprehensive guidelines to Hackintosh laptops. His method make this model works almost perfect.
- Thanks to Jaymonkey for his iDiot's Guide to Lilu and its Plug-ins.


