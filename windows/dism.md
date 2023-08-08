---
description: Utilizing Microsoft's Deployment Image Servicing and Management tool
---

# DISM

Deployment Image Servicing and Management (DISM.exe) is a command-line tool that can be used to service and prepare Windows images, including those used for Windows PE, Windows Recovery Environment (Windows RE) and Windows Setup. DISM can be used to service a Windows image (`.wim`) or a virtual hard disk (`.vhd` or `.vhdx`).

DISM can be used to mount and service a Windows image from a `.wim` file, `.ffu` file, `.vhd` file, or a `.vhdx` file and also to update a running operating system. It can be used with older Windows image files (`.wim` files). However, it cannot be used with Windows images that are more recent than the installed version of DISM.

Complete documentation can be found at [Microsoft](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/what-is-dism).

## Usage

DISM can be executed from the command line. The below example adds some drivers to an image stored on the machine:

```
DISM.exe /image:"c:\images\Image1" /Add-Driver /ForceUnsigned /DriverName:"C:\Drivers\1.inf" /DriverName:"C:\Drivers\2.inf" /DriverName:"C:\Drivers\3.inf"
```

Packages are installed in the order that they are listed in the command line. In the above example, `1.inf`, `2.inf`, and `3.inf` will be installed in the order in which they are listed in the command line.

### Installation

DISM can also be used to install software on the machine.&#x20;

For example, the command to install the Windows Telnet client would look like:

```powershell
PS C:\Windows\system32> dism /online /Enable-Feature /FeatureName:TelnetClient
```

The `/online` option of the DISM command allows users to perform actions on a running Operating System image.

It should be noted for the example above, installing Telnet requires administrative privileges, which could present challenges if running as a low-privilege user.
