# Per-Monitor (v2) High DPI support in WPF and WinForms applications

Starting with .NET Framework 4.7, WPF and WinForms support the new Per-Monitor (v2) dynamic DPI support. The new feature needs opt-in.

## WPF on .NET Framework 4.7+ and WinForms on .NET Core 3.0+

Add the following to your app's manifest:

```
<application xmlns="urn:schemas-microsoft-com:asm.v3">
    <windowsSettings>
        <dpiAwareness xmlns="http://schemas.microsoft.com/SMI/2016/WindowsSettings">PerMonitorV2, PerMonitor, System</dpiAwareness>
        <dpiAware xmlns="http://schemas.microsoft.com/SMI/2005/WindowsSettings">true/PM</dpiAware>
    </windowsSettings>
</application>
```

## WinForms on .NET Framework 4.7 - 4.8

In `manifest`, uncomment the following:

```
<compatibility xmlns="urn:schemas-microsoft-com:compatibility.v1">
  <application>
    <!-- Windows 10 compatibility -->
    <supportedOS Id="{8e0f7a12-bfb3-4fe8-b9a5-48fd50a15a9a}" />
  </application>
</compatibility>
```

Add the following in `app.config`:

```
<System.Windows.Forms.ApplicationConfigurationSection>
  <add key="DpiAwareness" value="PerMonitorV2" />
</System.Windows.Forms.ApplicationConfigurationSection>
```

## References

* [WPF-Samples/PerMonitorDPI at master · microsoft/WPF-Samples](https://github.com/microsoft/WPF-Samples/tree/master/PerMonitorDPI)
* [windows - Scaling the non-client area (title bar, menu bar) for per-monitor high-DPI support - Stack Overflow](https://stackoverflow.com/questions/36864894/scaling-the-non-client-area-title-bar-menu-bar-for-per-monitor-high-dpi-suppo)
* [High DPI support in Windows Forms | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/framework/winforms/high-dpi-support-in-windows-forms)
* [High DPI Desktop Application Development on Windows - Windows applications | Microsoft Docs](https://docs.microsoft.com/en-us/windows/desktop/hidpi/high-dpi-desktop-application-development-on-windows)