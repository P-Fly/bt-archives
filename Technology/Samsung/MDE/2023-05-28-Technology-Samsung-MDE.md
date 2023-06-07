# Samsung MDE

## 运行场景

 1. **Easy Pairing Scenarios**：连接到一个新的手机无需重复整个配对过程.

 2. **MDE Scenarios**：

    - **Connection Shift**：从手机到电视一键切换蓝牙耳机的连接。

    - **Paired Sync**：通过云端共享蓝牙配对列表。

 ![MDE Scenarios][1]

## Easy Pairing 强制要求

### 广播

 1. 地址类型：可以使用 **Public address** 或 **Static Random address**。

 2. 符合三星广播包格式：

    - **Flags**：

        1. **Limited Discoverable Mode**

        2. **General Discoverable Mode**

    - **Complete Local Name**

    - **Service Data** - **16 bit UUID**

        1. **Little Endian**。

        2. **Manufacturer ID**：**0xFDDB**。

        3. 由 **Adv data** 和 **Scan response** 组成。

        4. 数据格式如下：

            | 0 - 8 | 9 | 10 - 15 | 16 - 17 | 18 - 19 | 20 - 21 | 22 | 23 | 24 | 25 - 30 | 31 - 34 | 35 |
            | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
            | Samsung Packet General Info | Connection Status And Account | IC Packet Info | Account Hash1 | Account Hash2 | Account Hash3 | IC ID 1 | IC ID 2 | IC ID 3 | BT MAC Address | Battery Info | CRC |

            - Index **0 - 8**

                | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
                | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
                | 0x42 | 0x09 | 0x81 | 0x02 | 0x14 | Device Type Index | Icon Index | 0x21 | 0x01 |

            - Index **9**

                | | | | |
                | :-: | :-: | :-: | :- |
                | \[7:6\] | Connection Status | 0b00 | 00: disconnected <br> 01: connected <br> 11: fully connected state (No more connection available) <br> 10: reconnection failure |
                | \[5:3\] | Max account number | 0b001 | The number of devices which can be registered simultaneously. |
                | \[2:0\] | Account count | 0b000 | The number of accounts that is registered. |

            - Index **10**

                | | | | |
                | :-: | :-: | :-: | :- |
                | \[7:0\] | Sequence Number | 0x00 | Increase 1 every "power on" or "pairing mode" action. |

            - Index **11 - 15**

                | 11 | 12 - 13 | 14 | 15 |
                | :-: | :-: | :-: | :-: |
                | 0x17 | Device ID | 0x01 | 0x3C |

            - Index **16 - 24**

                | 16 - 17 | 18 - 19 | 20 - 21 | 22 | 23 | 24 |
                | :-: | :-: | :-: | :-: | :-: | :-: |
                | Account hash 1 | Account hash 2 | Account hash 3 | IC device ID 1 | IC device ID 2 | IC device ID 3 |

            - Index **25 - 34**

                | | | | | |
                | :-: | :-: | :-: | :-: | :- |
                | 25 - 30 | - | BT Mac Addr | - | Bluetooth Classic Mac Address XOR with BLE address for security reason. |
                | 31 | \[7\] | Battery Precision | 0b1 | 0: Can be express value in % <br> 1: Cannot be express value in %, so express approximate value in %. |
                | | \[6:0\] | Battery Level – case or solo device | 0 | If 1 in battery precision, then % value will not be display <br> 0x7F if not support |
                | 32 | \[7\] | Battery Precision | 0b1 | 0: Can be express value in % <br> 1: Cannot be express value in %, so express approximate value in % |
                | | \[6:0\] | Battery Level – left | 0 | If 1 in battery precision, then value will not be display <br> 0x7F if not support |
                | 33 | \[7\] | Battery Precision | 0b1 | 0: Can be express value in % <br> 1: Cannot be express value in %, so express approximate value in % |
                | | \[6:0\] | Battery Level – right | 0 | If 1 in battery precision, then value will not be display <br> 0x7F if not support |
                | 34 | \[7:0\] | - | 0x00 | Mandatory value is 0x00 |

    - 广播细节

        1. 时间间隔小于 **40ms**。

        2. Tx Power：1dbm。

        3. 广播在开机或配对模式下启用。

        4. 广播应该开启 **1min**。

        5. 当用户在广播时关闭耳机电源，**Connection Status** 必须更改为 **Fully Connected** 几秒，才能关闭附近设备的弹出窗口。

### 连接后的需求

 1. **Initial State**：注册 **UUID**(*a23d00bc-217c-123b-9c00-fc44577136ee*) 到 **SDP** 服务器以便进行 **SPP** 连接。

 2. **Connecting State**：

    - 检查手机的可用性：

        1. **SDP** 后检查 **UUID**(*a23d00bc-217c-123b-9c00-fc44577136ee*)。

        2. 如果 **UUID** 存在，耳机应该等待 **SPP**，并且 **ACCOUNT_HASH** 和 **IC_DEVICE_ID** 将通过 **SPP** 连接进行更新：

            - 如果手机是同一个帐户，那么相同的 **ACCOUNT_HASH** 和 **IC_DEVICE_ID** 将被覆盖。

            - **ACCOUNT_HASH** 是由三星帐户的电子邮件地址生成的。

            - **IC_DEVICE_ID** 是由 **Source Device** (*Phone, TV, Refrigerator*) 生成并且发给耳机的。

        3. 如果UUID不存在：

            - 耳机将更新 **ACCOUNT_HASH** 和 **IC_DEVICE_ID** 为默认值(0x00)。

            - 耳机将 **Connection State** 设为 **fully connected state**。

    - **SPP Connection**：

        1. 如果手机支持，那么手机将通过 **UUID**(*a23d00bc-217c-123b-9c00-fc44577136ee*) 连接到耳机上。

        2. 连接流程：

            ![SPP Connection Sequence][2]

 3. **SPP** 数据格式：

    - **Phone Side**：

        | 0 | 1 | 2 | 3 | 4 | 5 - 6 | 7 |
        | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
        | 0xFF | 0x00 | 0x13 | 0x03 | IC ID | Account Hash | 0xFE |

    - **Headset Side**：

        | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
        | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
        | 0xFF | 0x01 | 0xFD | 0x02 | 0x13 | Result | 0xFE |

## MDE 强制要求

### 连接要求

 1. 当由于 **REMOTE USER TERMINATED CONNECTION (0x13)** 的原因断开连接时，耳机应进入 **Page Scan** 模式。

 2. 耳机应接受来自 **Page Scan** 前未连接的任何设备的连接请求（至少 1min）。

### 连接后的要求

 1. 注册 **UUID**(*a23d00bc-217c-123b-9c00-fc44577136ee*) 到 **SDP** 服务器以便进行 **SPP** 连接。

 2. **SPP Connection**。

    - SPP连接由远端设备发起。

    - 连接流程：

        ![SPP Connection Sequence][3]

 [1]: ./images/MDE_scenarios.jpg
 [2]: ./images/easy_pairing_spp_connection_sequence.jpg
 [3]: ./images/connection_shift_spp_connection_sequence.jpg
