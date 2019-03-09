# Hyper-V Notes

## 1. Nested Virtualization
As of 2019-03-07, Windows 10 19H1 Build 18351, there is no GUI for Nested Virtualization. Nested Virtualization on AMD processors is not supported by Hyper-V, but available on other hypervisors.

To enable Nested Virtualization for a VM, open Powershell as admin and execute:
```powershell
> Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```

## 2. About GPU
_Written on 2019-03-10_

On Windows Server 2019 and Windows 10 Version 1809, `RemoteFX` is deprecated. New technologies such as `GPU-P` and `GPU-PV` are still in development and not _officially_ available, as of 2019-03-07. However, RemoteFX is still available through `Powershell`.

On Windows 10 Version 1903, Microsoft added **Windows Sandbox**, which uses **vGPU** by default. There is no official documentation mentioning the technology behind it. But it explicitly mentioned a requirement of __WDDM 2.5 or higher__, which happens to be the minimal requirement for `GPU-P`.

So, I did some experiments on Windows 10 19H1 Build 18351. First get a list of available Powershell command for Hyper-V:
```powershell
> Get-Command -Module Hyper-V
```
Then I noticed a few new commands. These new commands are available starting from Windows 10 1809, but there is no documentation.
```powershell
Add-VMGpuPartitionAdapter
Get-VMGpuPartitionAdapter
Get-VMPartitionableGpu
Remove-VMGpuPartitionAdapter
Set-VMGpuPartitionAdapter
Set-VMPartitionableGpu
```
`Get-VMPartitionableGpu` returns my dGPU with a WDDM 2.5 driver.

I attempted to configure GPU-P (reference 4):
```powershell
# Allow guest to control cache types for MMIO
Set-VM -GuestControlledCacheTypes $true -VMName $vm
# Set lower/upper MMIO spaces
Set-VM -LowMemoryMappedIoSpace 1GB -VMName $vm
Set-VM -HighMemoryMappedIoSpace 32GB -VMName $vm

# Add the GPU adapter
Add-VMGpuPartitionAdapter -VMName $vm 
```
The target VM runs Windows 10 20H1 Build 18850. It detected the virtualized GPU as `Microsoft Virtual Renderer` and loaded an inbox driver `vrd.sys`. But the driver seemed to have caused a `LiveKernelEvent` and was disabled soon after boot.

Also, there is no documentation regarding the `LowMemoryMappedIoSpace` and `HighMemoryMappedIoSpace` parameter. I can't figure out their meaning and purpose.

Although my little attempt failed, I look forward to using `GPU-P` to boost VM performance.

Refer to:
* [How to configure RemoteFX using Powershell on Windows Server 2019](https://techcommunity.microsoft.com/t5/Windows-Server-for-IT-Pro/Server-2019-Hyper-V-VM-using-GPU/td-p/303761)
* [RemoteFX vGPU -> GPU-P -> GPU-PV](https://www.brianmadden.com/opinion/RemoteFX-vGPU-put-out-to-pasture-as-Microsoft-RDP-grows-up)
* [Hybrid track: Remote Desktop Services on-premises and on Azure](https://www.microsoft.com/en-us/cloud-platform/windows-server-summit#hybridtrack)
* [Describes how to partition and assign a GPU to several VMs through Hyper-V](https://gist.github.com/cwilhit/30faa5cb47fa0ad81053db29e42793f4)