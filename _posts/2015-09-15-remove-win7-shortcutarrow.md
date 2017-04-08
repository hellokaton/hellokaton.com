---
layout: post
title: 去除win7快捷方式小箭头
categories: win7
---

```bash
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons" /v 29 /d "%systemroot%\system32\imageres.dll,196" /t reg_sz /f
taskkill /f /im explorer.exe
attrib -s -r -h "%userprofile%\AppData\Local\iconcache.db"
del "%userprofile%\AppData\Local\iconcache.db" /f /q
start explorer
```

将上面的代码复制到记事本中，然后另存`123.bat`，文件类型为所有文件，然后右键123.bat-以管理员身份运行，注意代码格式不要复制错误
