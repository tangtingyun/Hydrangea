---
title: android异常记录
tags: android
---
- 记录平时遇到到android异常错误(持续更新...)
<!-- more -->
1. not attached to window manager
    在dismissProgressDialog的时候 报错 因为宿主context 已经销毁了
    解决办法：[stackoverflow](https://stackoverflow.com/questions/22924825/view-not-attached-to-window-manager-crash)
    ```
        // 参考上面答案 觉得这种处理比较好
        ProgressDialog myDialog = new ProgressDialog(getActivity());
        myDialog.setOwnerActivity(getActivity());
        ...
        Activity activity = myDialog.getOwnerActivity();
        if( activity!=null && !activity.isFinishing()) {
            myDialog.dismiss();
        }
    ```