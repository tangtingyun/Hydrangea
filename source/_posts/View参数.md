---
title: View参数
date: 2019-10-21 20:54:15
tags: android-view
---
encapsulates 封装

view 测量参数记录
<!-- more -->
1. **`MeasureSpec`**

   - `UNSPECIFIED`

    The parent has not imposed any constraint on the child. It can be whatever size it wants.
   - `EXACTLY`

    The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.

   - `AT_MOST`

    The child can be as large as it wants up to the specified size.

`wrap_content` -> `MeasureSpec_AT_MOST`

`match_parent` -> `MeasureSpec_EXACTLY`

`具体值` -> `MeasureSpec.EXACTLY`

2. **`ViewManager`**

```sequence
Title:add view
viewManager->windowManager:addview
windowManager->WindowManagerImpl:addview
WindowManagerImpl->WindowManagerGlobal:addview
```
```sequence
WindowManagerGlobal->ViewRootImpl:
Note over ViewRootImpl:setview
Note over ViewRootImpl:requestLayout
```
