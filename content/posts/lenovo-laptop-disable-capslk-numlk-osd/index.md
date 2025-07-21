+++
date = "2024-01-14T14:25:00+08:00"
# lastmod = "2025-07-21T21:18:33+08:00"
draft = false
title = "Lenovo 笔记本关闭数字锁定和大写锁定提醒"
# summary = ""
+++

在对应注册表位置添加或修改成如下值

```reg
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LenovoFnAndFunctionKeys\VantageToast]

"ShowCapslkOSD"=dword:00000000
"ShowNumlkOSD"=dword:00000000
```
