# Workaround: On Windows 10 Version 1903, `dwm.exe` stuck in a single-thread loop after disconnecting from an RDP session

Use the following `Powershell` command to check CPU usage:
```powershell
Get-Counter '\Process(*)\% Processor Time' | Select-Object -ExpandProperty countersamples | Select-Object -Property instancename, cookedvalue| Sort-Object -Property cookedvalue -Descending| Select-Object -First 20| ft InstanceName,@{L='CPU';E={($_.Cookedvalue/100).toString('P')}} -AutoSize
```

Then run `Get-Process` to get the PID of `dwm.exe`.
```powershell
Get-Process dwm
Stop-Process -id 12345
```