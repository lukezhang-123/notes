

- powershell 脚本参考， 克隆hyper-v vm

https://gist.github.com/aaronNGi/3447525bac2cbad50669a6716d1073f2

- clone hyper-v vm, export->import->delete old export

```
$SourceVMName = "PC0001"
$CloneVMName = "PC0001-CLONE"
$ExportFolder = "E:\Export"
$CloneFolder = "F:\VMs\$CloneVMName"

If (Test-Path $CloneFolder){
    Write-Warning "Clone folder: $CloneFolder already exists. Aborting script..."
    Break
}

# Export the Source VM
Export-VM $SourceVMName -Path $ExportFolder

# Import the Exported VM, full copy, and generating a new ID
$CloneVMConfigFile = (Get-ChildItem "$ExportFolder\$($ReferenceVM.Name)\Virtual Machines" -Filter *.vmcx -Recurse | Select -First 1).Fullname

$CloneVMConfig = @{
    Path = $CloneVMConfigFile;
    SnapshotFilePath = Join-Path $CloneFolder "Snapshots";
    VhdDestinationPath = Join-Path $CloneFolder "Virtual Hard Disks";
    VirtualMachinePath = $CloneFolder;
}

$Result = Import-VM -Copy -GenerateNewID @CloneVMConfig 

# Rename the imported VM (will be imported with original name)
$Result | Rename-VM -NewName $CloneVMName

# Remove the exported VM
$SourceVMExportPath = "$ExportFolder\$SourceVMName"
If (Test-Path $SourceVMExportPath) { Remove-Item -Path $SourceVMExportPath -Recurse -Force }
```