# Configure OpenSSH on Windows
_Written on 2019-03-12._

Windows 10 has integrated __OpenSSH client and server__ as an _optional feature_. It also has an OpenSSH agent that can store and manage private keys. In this guide, we will turn on OpenSSH server and `ssh-agent` service, and use OpenSSH client to connect to an ssh server.

## Enable the Optional Feature
On Windows 10, go to _Settings -> Apps -> Apps & Features -> Optional Features_, click _Add a Feature_, select _OpenSSH Client_ and _OpenSSH Server_, and install them.

If you can't find any _Optional Features_, check your Windows 10 version. Newer versions may have different entries. Older versions didn't include these Optional Features.

## Configure the client and the server
The client configuration file is located in `~/.ssh/`, the same path as on Linux. However, the server configuration is in `C:\ProgramData\ssh\`.

_Note that as of Windows 10 1809, the integrated OpenSSH server doesn't support public key authentication. Only password authentication works._

If the server listening port is changed, you have to create new firewall rules to allow the new port. The pre-defined rule cannot be changed.

## Enable and start the service.
Open _Services_, enable and start _OpenSSH SSH Server_, or `sshd`, and _OpenSSH Authentication Agent_, or `ssh-agent`.

Or use `powershell`:
```powershell
Set-Service -Name sshd -StartupType Automatic -Status Running
Set-Service -Name ssh-agent -StartupType Automatic -Status Running
```

## Add your private key to `ssh-agent`
```bash
$ ssh-add ~/.ssh/id_ed25519
```
Use `ssh-add -l` to view added private keys.