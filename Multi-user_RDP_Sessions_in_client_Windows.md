# Multi-user RDP Sessions in Windows Client Editions

On client Windows editions like Windows 10, concurrent user sessions through RDP are supported but disabled. To enable it, make small changes to `termsrv.dll`.

## Prepare a new `termsrv.dll`
Copy `C:\Windows\System32\termsrv.dll` to a working directory where you will be performing modifications to the binary file. Make a backup of the file if necessary.

Use a Hex Editor to open `termsrv.dll`.

Search for Hex values of `39 81 3C 06 00 00 0F 84`.

* On Windows 10 1809, it should be like `39 81 3C 06 00 00 0F 84 7F 2C 01 00`.
* On Windows 10 1903, it should be like `39 81 3C 06 00 00 0F 84 5D 61 01 00`.

Replace it with `B8 00 01 00 00 89 81 38 06 00 00 90`.

## Modify file permissions
Take ownership of `C:\Windows\System32\termsrv.dll` and add necessary permissions for yourself.

## Apply changes
Stop `TermService` service. Replace `C:\Windows\System32\termsrv.dll` with your new `termsrv.dll`. Start the service. Now multiple users should be able to log in and use the system simultaneously through RDP.

## References
* [Multiple RDP (Remote Desktop) sessions in Windows 10](https://www.mysysadmintips.com/windows/clients/545-multiple-rdp-remote-desktop-sessions-in-windows-10)