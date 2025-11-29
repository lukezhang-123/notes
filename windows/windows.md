- win11 鼠标右键直接显示更多选项

```
管理员打开cmd
reg.exe add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f
taskkill /F /IM explorer.exe & start explorer
```

- 删除恢复分区

```
管理员打开cmd
reagentc /info  # 位置 harddisk5\partition4
reagentc /disable

diskpart
list disk
select disk 5
detail disk  # 确认磁盘信息
list partition
select partition 4  # 类型 恢复
delete partition override
```

- 移动恢复分区到磁盘尾，dism制作分区镜像后应用到新分区，reagentc注册新的恢复分区

https://superuser.com/questions/1453790/how-to-move-the-recovery-partition-on-windows-10

```
复制恢复分区的文件到新的盘尾分区，diskgenius,工具，克隆分区，文件复制

diskpart 设置新恢复分区属性
set id=de94bba4-06d1-4d40-a16a-bfd50179d6ac override
gpt attributes=0x8000000000000001

reagentc /disable
Reagentc /setreimage /path S:\Recovery\WindowsRE
reagentc /enable
reagentc /info
```

- powershell升级，打开powershell

```
当前版本
$PSVersionTable.PSVersion
查看可用版本
winget search Microsoft.PowerShell
安装新版本，从winget源安装Microsoft.Powershell
winget install --id Microsoft.Powershell --source winget

随便目录里，右键，在终端中打开，上面最后向下箭头，设置，左边导航，启动，右边最上面第一个，默认配置文件，下拉选择，powershell，原来的默认是windows powershell，重新打开powershell，第一行显示新版本号表示修改默认powershell为最新版成功
```

- powershell修改类似ps1提示符

```
用vscode打开profile文件，C:\Users\xxx\Documents\PowerShell\Microsoft.PowerShell_profile.ps1
code $PROFILE
```

```
function prompt {
    $dateTime = get-date -Format "yyyy.MM.dd HH:mm:ss"
    $currentDirectory = $(Get-Location)

    write-host "$dateTime" -NoNewline -ForegroundColor green
    # Convert-Path needed for pure UNC-locations
    write-host " $(Convert-Path $currentDirectory)" -ForegroundColor gray
    write-host ">" -NoNewline -ForegroundColor Yellow
    return " "
}
```

- 运行 netplwiz 创建用户

- windows11 使用Microsoft账户不能远程桌面RDP

```
确定端口 connected
curl -kv ip:3389

设置（Win + I），账户 > 登录选项
关闭“为了提高安全性，仅允许对此设备上的 Microsoft 账户使用 Windows Hello 登录（推荐）”
重启，使用Microsoft账户密码登录一次
尝试RDP登录
开启  Windows Hello 登录
```


