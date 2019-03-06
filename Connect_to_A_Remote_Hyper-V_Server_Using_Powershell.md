# How to Connect to A Remote Hyper-V Server Using Powershell

*First edition 2018-11-18: This article should apply to any client or server version of Windows newer than Build 17134. The Hyper-V server may run on either a Windows Server or a Windows 10 PC. The client should be a Windows 10 PC.*

*Updated 2019-03-06: Migrated to markdown and changed some wording.*

## 1. Preparation
Make sure the Hyper-V server is running and Hyper-V manager is installed on the client.
## 2. Configure the server to allow incoming `PSRemoting` connections
On the server, run the following commands in `Powershell`:
```powershell
Enable-PSRemoting
Enable-WSManCredSSP -Role server
```
This won't automatically enable the corresponding firewall rules(might be a bug). Go to `Advanced Firewall Settings` and enable the following rules: `Windows Remote Management (HTTP-In)`

The Compatibility mode rules are not needed. Inbound connections are going through port `5985`. External network gateway might need to be configured too. The default setting is allow `Local Subnet` access only. You may want to remove the restriction to allow all IP addresses.

Now that we have finished server configuration, here comes the tricky part - the client.
## 3. Begging (literately) the client to be willing to connect
Looks like the programmers working on this at Microsoft really have severe trust issues. We are having great trouble here trying to make the client trust the host we want to connect.

First you have manually start the `WinRM` service, since PowerShell will fail to keep the service running for some reason and fail to work(Probably a bug). Just make sure the service is running. Then run the following in PowerShell:
```powershell
Enable-PSRemoting
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "homeserver"
Enable-WSManCredSSP -Role client -DelegateComputer "homeserver"
```
The hostname can either be NetBIOS name or IP address.

Now we need to change the credential delegation settings on your PC to allow for the `non-domain NTLM authentication`.

Navigate through `Computer Configuration`, `Administrative Templates`, `System`, `Credential Delegation`, and right click on "Allow delegating fresh credentials with NTLM-only server authentication" and select Edit.

Enable the rule and type in `Wsman/homeserver` (remember to replace "homeserver" with the name of your Hyper-V host)
## 4. Connect
Use the Hyper-V Manager to connect to the remote server. You might need to select the "Connect as another user" box for domain reasons or whatever.

Note that connecting to VMs are not supported. Anything other than that should work, like starting a VM.

## References
1. [`Enable-PSRemoting`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-6)

2. [2.	Remotely manage Hyper-V Hosts | Microsoft Docs.](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/remotely-manage-hyper-v-hosts)

3. [3.	Remotely manage a Non-Domain Hyper-V host from Windows 10.](https://tweaks.com/windows/67216/remotely-manage-a-nondomain-hyperv-server/)
