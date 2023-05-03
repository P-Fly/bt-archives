# 在 BT Settings 中设备名显示了两次

## 问题描述

最近在调试一个 **Dual Mode Device** 时，发现在 **安卓** 下的 **BT Settings** 中会出现两个一模一样的设备（**iPhone** 没有这个问题）。

 ![Two names in the BT settings][1]

## 问题原因

上图中第二个 **My Device** 是期望的 **BT Device**。

第一个 **My Device** 其实是通过广播识别到的 **BLE Device**。

之所以被显现出来，是因为在我的广播包中存在广播数据 **02 01 0A**。其中 **Bit 1** 把该设备错误的标识为 **LE General Discoverable Mode**。将广播包改为 **02 01 08** 后，第二个 **My Device** 就不会出现。

 ![CSS 1.3 Flags][2]

 [1]: ./images/two_names_in_the_BT_settings.jpg
 [2]: ./images/CSS_1.3_Flags.jpg
