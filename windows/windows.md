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

