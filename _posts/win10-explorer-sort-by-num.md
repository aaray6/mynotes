---
title: win10_explorer_sort_by_num
date: 2019-03-20 09:23:26
tags:
---

Win10资源管理器中文件名排序中，数字部分是把数字装换为数值后进行排序

比如

```text
201801AAA
201901BBB
20180101CCC
```

而我们通常是希望如下排序

```text
201801AAA
20180101CCC
201901BBB
```

解决方法

> [Is there a way to fix the sort by name in Windows 10 File Explorer?](https://superuser.com/questions/1065671/is-there-a-way-to-fix-the-sort-by-name-in-windows-10-file-explorer)

- In the start menu, search for "Edit Group Policy" or "gpedit.msc" and press enter.
In the Local Group Policy Editor, navigate to User Configuration / Administrative Templates / Windows Components / File Explorer.
- In Windows 10 this is in File Explorer, whereas Windows 7 apparently called it Windows Explorer.
- Double-click Turn off numerical sorting in File Explorer and select Enabled, then click OK.

或者安装FreeCommander

> [FreeCommander Sorting Setting](https://freecommander.com/fchelpxe/en/Sorting.html)

Under "Tools → Settings → View → File/folder list" in the tab "Sorting" the following settings are offered.

- Alphabetical - string sort
- Alphabetical - word sort
- Natural sorting (as Explorer)
