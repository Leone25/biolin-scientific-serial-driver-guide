# Biolin Scientific Serial Driver Guide
A guide to modify the usb serial driver from FTDI to work with Biolin Scientific DipCoater (0403:ef93)

## Background
The device uses a chip from FTDI (0403) that is customized to not be recognized by the publicaly available drivers, rather requiring you to purchase them from Biolin Scientific directly.
As it turns out, the drivers have nothing special, and just rely on windows not recognizing the specific product id.
To work around this we have to modify the public driver and the sign it again so that windows recognizes it.

## How
1. Download and install Windows SDK and WDK following step 2 and 3 of this [guide](https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk), be aware that depending on when you install them, the file paths might change slightly from version to version, so before running any of the next commands you might have to change them slighly.
2. Go to [the download page for FTDI drivers](https://ftdichip.com/drivers/d2xx-drivers/) and in the table download the latest `Windows (Desktop)` driver (x86 or x64 doesn't matter), and extract it somewhere easy, for this example, I'll create a temp called `C:/moddriver`
4. Inside `C:/moddriver` delete the `dtdibus.cat` file
5. In the same folder, open the `ftdibus.inf` file in notepad
6. Find the section titled `[FtdiHw]`, and at the bottom, copy and paste the last line and edit both of the `0000` to be `EF93`, do the same for the section `[FtdiHw.NTamd64]` and again for the `[Strings]` section (only one `0000` to replace here)
7. Open a cmd window and run the following commands one line at a time:
   ```sh
   cd C:/moddriver
   $todaydate = Get-Date
   $add99year = $todaydate.AddYears(99)
   $cert = New-SelfSignedCertificate -Subject "WOSHUB” -Type CodeSigningCert -CertStoreLocation cert:\LocalMachine\My -notafter $add99year
   $CertPassword = ConvertTo-SecureString -String “P@ss0wrd” -Force –AsPlainText
   Export-PfxCertificate -Cert $cert -FilePath C:\DriverCert\myDrivers.pfx -Password $CertPassword
   $certFile = Export-Certificate -Cert $cert -FilePath C:\DriverCert\drivecert.cer
   Import-Certificate -CertStoreLocation Cert:\LocalMachine\AuthRoot -FilePath $certFile.FullName
   Import-Certificate -CertStoreLocation Cert:\LocalMachine\TrustedPublisher -FilePath $certFile.FullName
   "C:\Program Files (x86)\Windows Kits\10\bin\10.0.16299.0\x86\inf2cat.exe" /v /os:XP_X86,Vista_X86,Vista_X64,7_X86,7_X64,8_X86,8_X64,6_3_X86,6_3_X64,10_X86,10_X64 /driver:.
   "C:\Program Files (x86)\Windows Kits\10\bin\x86\signtool.exe" sign /tr http://timestamp.digicert.com /td SHA256 /v /f C:\moddriver\myDrivers.pfx /p P@ss0wrd C:\moddriver\ftdibus.cat
   ```
8. Lastly, in file explorer, you can right click on the `ftdibus.inf` and select `install`
9. If you plug in the device it should now be recognized and in device manager it should show up under com devices

## Notes
- This guide have been simplified a lot, there are no explanations on what you are doing, if you are interested in more info, look at the links in the credits
- Similar devices might have the same issue but different hardware id, you can follow the same guide but replace `0000` with a different hardware id (all upper-case) which can be found with [usbview from ftdi's website](https://ftdichip.com/utilities/#microsoft-usbview), when the device is plugged in it should show up with a warning symbol

## Credits
- https://woshub.com/how-to-sign-an-unsigned-driver-for-windows-7-x64/
- https://learn.adafruit.com/how-to-sign-windows-drivers-installer/signing-driver
