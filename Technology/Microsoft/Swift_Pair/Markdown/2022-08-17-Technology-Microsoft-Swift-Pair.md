# Swift Pair

## 简介

**Swift Pair** 是在 **Windows 10** 的 **1803** 版本中引入的，是将蓝牙外设与电脑配对的最新方法。

作为配对的下一个演变，用户不再需要通过游览 **Settings App** 找到要配对的外围设备。当附近出现新的外设并且该设备准备就绪时，**Windows** 就会弹出一条通知并请求连接。

此功能使用的步骤非常简单：

 1. 将蓝牙外设置于配对模式。

 2. 当外围设备靠近时，**Windows** 将向用户显示通知。

 3. 选择 **连接** 将开始配对外设。

 4. 当外围设备不再处于配对模式或不再靠近时，**Windows** 将从操作中心删除通知。

详情可以参见链接：[bluetooth-swift-pair][1]

## 配对流程

 ![Swift Pair Flow][2]

## 外设行为

发现 **Swift Pair peripherals** 是通过 **BLE** 协议，需要使用 **LE advertisements**：

 - **Quickest Discovery** 阶段：信标每 **30ms** 一次，持续大于 **30S**。

 - **Normal Cadence** 阶段：信标每 **100ms** 或 **152.5ms** 一次。

 - **Cool-down** 阶段：在退出配对模式前 **30S** 停止广播供应商信息。

如果外设已经没有可用配对，则删除最先连接的设备。

## 有关 Swift Pair 通知的外设信息

 - 外设需要定义 **Class of Device (CoD)** 或 **Peripheral Name**，并包含在同一广播包中。

 - 由于电源和隐私问题，**Windows** 不会主动扫描。因此，外设信息无法存储在 **Scan Response** 中。

 - 外观设置：

    - **Bluetooth LE only**：广播包中的 **Appearance Type** 会被用于定义 **CoD**，并用于显示图标。

    - **Dual mode peripheral**：**Swift Pair** 中已经包含了 **CoD**，是一个 3字节 的值。参看[Assigned Numbers for Baseband][3]

    - 如果检测到 **CoD**，则显示的图标与设置中显示的图标相同。

    - 如果未检测到 **CoD**，则 **Windows** 默认使用蓝牙图标。

 - 名称设置：

    - 推荐使用 **Bluetooth friendly name section** 或者使用 **Swift Pair** 中定义的 **Display Name** 区域。

    - 如果检测到名称，将显示 **New [Peripheral Name] found**。

    - 如果未检测到名称，通用的名称将被显示 **New Bluetooth mouse found**，**New Bluetooth headphones found** 或 **New Bluetooth**。

## Swift Pair 所需的规格功能

 - 如果外设在没有任何显式用户操作的情况下为 **Swift Pair** 发出信标，则支持 **LE Privacy**。

 - 如果支持 **LE Privacy**，则外设应在 **Swift Pair** 会话期间暂停 **Rotating the Bluetooth LE Address**。

    *Note：Windows会将轮换地址认为是新的设备请求，并为单个外围设备显示两条通知。因此在 Cool-down 阶段完成之前，都不应更改设备地址。*

 - 如果双模外设希望同时通过 **BR/EDR** 和 **LE** 进行配对，则该外设必须支持两种协议的安全连接。**Windows** 首先通过 **LE** 配对，然后得到 **BR/EDR** 的密钥进行安全连接。在不使用安全连接的情况下，不支持 **Paring over Bluetooth LE and BR/EDR** 的方式进行配对。

    *Note：这一步应该是需要使用CTKD的方式交换 LE 和 BR/EDR 的秘钥。*

## 推荐经验

 - 当外围设备第一次通电时，进入配对模式。

 - 不要无限期地广播，**Windows** 在每个会话期间只显示一个通知。

 - **Dual mode peripheral** 通过 **Paring over Bluetooth LE and BR/EDR with Secure Connections** 可以节省负载空间。

## 广播包数据结构

 ![Payload Structures][4]

 [1]: https://docs.microsoft.com/en-us/windows-hardware/design/component-guidelines/bluetooth-swift-pair
 [2]: ./images/swift_pair_flow.jpg
 [3]: https://btprodspecificationrefs.blob.core.windows.net/assigned-numbers/Assigned%20Number%20Types/Baseband.pdf
 [4]: ./images/swift_pair_payload_structures.jpg
