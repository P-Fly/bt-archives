# 设备地址

蓝牙设备使用 **设备地址** 和 **地址类型** 来识别设备，其中地址长度为 **48Bits**。

具体的 **地址类型** 如下：

 ![Device address type][1]

设备应使用至少一种类型的 **设备地址**，并且可以同时包含两种地址。

设备的 **Identity Address** 是它在传输的数据包中使用的 **Public Device Address** 或 **Random Static Device Address**。如果一个设备正在使用 **Resolvable Private Addresses**，那么它应该还有一个 **Identity Address**。

当比较两个 **设备地址** 时，也应同时比较 **地址类型**。如果两个设备的 **地址类型** 不同，即使他们的 **设备地址** 一样，也是不同的设备。

## Public Device Address

每个蓝牙设备都应分配一个唯一的 **48-bit** 蓝牙设备地址 **(BD_ADDR)**。

蓝牙设备地址的格式如下：

 ![Format of BD_ADDR][2]

通常 **NAP** 和 **UAP** 是生产厂商的唯一标识码，由蓝牙权威部门分配给不同的厂商。而 **LAP** 是在厂商内部自由分配。

蓝牙设备地址可以取任何值，除了 **Reserved Addresses** 中定义的：**LAP 0x9E8B00 ~ 0x9E8B3F**。

## Random Device Address

**Random Device Address** 可以分为以下几类：

 - **Static Device Address**

 - **Private Device Address**

    - **Non-resolvable Private Address**

    - **Resolvable Private Address**

具体的子类型通过地址的 **Bit[47:46]** 来表示。

 ![Sub-types of random device addresses][3]

### Static Device Address

 ![Format of static address][4]

该地址是一个 **48-bit** 随机生成的地址，并满足以下要求：

 - 地址的随机部分至少一位应为 **0**。

 - 地址的随机部分至少一位应为 **1**。

地址不能在运行期间改变，设备可以在每次上电时将该地址初始为一个新值。

如果设备的地址被更改，则存储在 **Peer devices** 中的地址将无效，并且无法使用旧地址进行回连。

### Private Device Address

#### Non-resolvable Private Address

 ![Format of non-resolvable private address][5]

该地址是一个 **48-bit** 随机生成的地址，并满足以下要求：

 - 地址的随机部分至少一位应为 **1**。

 - 地址的随机部分至少一位应为 **0**。

 - 该地址不等于 **Public Device Address**。

#### Resolvable Private Address

 ![Format of resolvable private address][6]

 1. 生成 **Resolvable Private Address**：

    - 为生成该地址，设备必须有本地或对端的 **Identity Resolving Key (IRK)**。

    - 该地址的随机数被称为 **prand**，并满足以下要求：

        - 地址的随机部分 **prand** 至少一位应为 **1**。

        - 地址的随机部分 **prand** 至少一位应为 **0**。

    - 该地址的 **hash** 由 **[Vol 3] Part H, Section 2.2.2** 中定义的随机地址函数生成：

        ```
        hash = ah(IRK, prand)
        ```

    - **prand** 和 **hash** 连接在一起，则可以生成完整的地址：

        ```
        RandomAddress = prand || hash
        ```

 2. 解析 **Resolvable Private Address**：

    - 从该地址中分别提取 **hash** 和 **prand**。

    - 通过 **[Vol 3] Part H, Section 2.2.2** 中定义的随机地址函数生成 **LocalHash**。

        ```
        LocalHash = ah(IRK, prand)
        ```

    - 如果 **LocalHash** 等于地址中的 **hash**，则可知该地址的设备为此 **IRK** 对应的设备，即完成了对发送设备的身份识别。

    - 如果存储了多个 **IRK**，需要使用每一个 **IRK** 重复解析过程，直到成功完成解析或所有 **IRK** 都已经被尝试。

 [1]: ./images/device_address_type.jpg
 [2]: ./images/format_of_BD_ADDR.jpg
 [3]: ./images/sub_types_of_random_device_addresses.jpg
 [4]: ./images/format_of_static_address.jpg
 [5]: ./images/format_of_non-resolvable_private_address.jpg
 [6]: ./images/format_of_resolvable_private_address.jpg
