- win11 鼠标右键直接显示更多选项

```
管理员打开cmd
reg.exe add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f
taskkill /F /IM explorer.exe & start explorer
```