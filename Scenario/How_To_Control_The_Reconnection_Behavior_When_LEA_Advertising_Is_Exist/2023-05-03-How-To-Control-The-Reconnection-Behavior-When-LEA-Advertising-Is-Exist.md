# 当 LEA 广播存在时，如何控制回连行为

## 问题描述

在 **LE Audio** 中，回连行为是由手机主动发起的，那怎么才能控制当前的回连行为？

因为某些场景下，**UI** 可能并不希望手机主动回连设备。比如早上带手机出门，下午用户回家后不一定希望回连早上的设备。

## 解决方法

在 **Basic Audio Profile** 中，已经定义了该场景。开发者可以通过 **Targeted Announcement** 标志来表明是否希望手机主动回连。

 - **Targeted Announcement == 1** 希望手机主动回连设备。

 - **Targeted Announcement == 0** 不希望手机主动回连设备，除非用户通过手机 **UI** 明确地表示需要连接到设备上。

 ![Targeted Announcement][1]

 [1]: ./images/Targeted_Announcement.jpg
