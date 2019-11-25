# Workaround: On Windows 10 Version 1903, `dwm.exe` stuck in a single-thread loop after disconnecting from an RDP session

Update: On October 24 2019, Microsoft finally released KB4522355 (Build 18362.449) to address the issue.

Use the following `Powershell` command to check CPU usage:
```powershell
Get-Counter '\Process(*)\% Processor Time' | Select-Object -ExpandProperty countersamples | Select-Object -Property instancename, cookedvalue| Sort-Object -Property cookedvalue -Descending| Select-Object -First 20| ft InstanceName,@{L='CPU';E={($_.Cookedvalue/100).toString('P')}} -AutoSize
```

Then run `Get-Process` to get the PID of `dwm.exe`.
```powershell
Get-Process dwm
Stop-Process -id 12345
```