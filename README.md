# Hackintosh for Lenovo Legion R7000P 2020H

## 简介

适用于联想拯救者R7000P（2020款）的预配置OpenCore EFI。

| Item                 | Info                                                       |
| -------------------- | ---------------------------------------------------------- |
| Opencore Version     | 0.9.7                                                      |
| Model                | 联想拯救者R7000P（2020款）<br />Lenovo Legion R7000P 2020H |
| SMBIOS used          | MacBookPro16,3                                             |
| Target MacOS Version | macOS Sonoma 14.3                                          |

本EFI还可能兼容以下型号，请自行测试可用性：

- Lenovo Legion 5-15ARH05H: Sharing same bios firmware with R7000P2020H, most likely compatible.
- 联想拯救者R7000（2020款）

## 可用性

状态标识：

- 🟢=可用
- 🟡=可用但存在问题（问题情况详见下文）	
- 🔴=不可用

### 硬件

| Item       | Info                        | Status                                                     | Notes                                                        |
| ---------- | --------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| CPU        | AMD Ryzen™ 7 4800H          | 🟢                                                          | 电源管理&传感器信息显示： [AMDRyzenCPUPowerManagement](https://github.com/trulyspinach/SMCAMDProcessor) + [SMCAMDProcessor](https://github.com/trulyspinach/SMCAMDProcessor) |
| 核显       | AMD Radeon™ Vega 8          | 🟡 硬件加速<br />🔴 视频硬解<br />🔴 视频输出（HDMI、C口DP）  | 驱动：[NootedRed](https://github.com/NootInc/NootedRed)<br />传感器信息显示：[RadeonSensor](https://github.com/aluveitie/RadeonSensor) + [SMCRadeonGPU](https://github.com/aluveitie/RadeonSensor)<br />视频输出独显直通，没有希望 |
| 独显       | Nvidia Geforce RTX 2060 6GB | 🔴                                                          | 使用[Bumblebee Method](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/laptop-disable.html#bumblebee-method)禁用，`SSDT-dGPU-OFF-NoHybGfx.aml` |
| 无线网卡   | Intel® Wi-Fi 6 AX200        | 🟢 WIFI<br />🟡 蓝牙                                         | 驱动：<br />[IntelBluetoothFirmware.kext + IntelBTPatcher](https://github.com/OpenIntelWireless/IntelBluetoothFirmware/pull/446) (使用最新PR，修复部分LE设备无法连接问题) <br />+ [AirportItlwm-Sonoma](https://github.com/OpenIntelWireless/itlwm/releases/tag/v2.3.0-alpha)<br />+ [BlueToolFixup](https://github.com/acidanthera/BrcmPatchRAM?tab=readme-ov-file#bluetoolfixupkext) |
| 有线网卡   | Realtek RTL8111             | 🟢                                                          | 驱动：[RealtekRTL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X) |
| 音频       | Realtek ALC257              | 🟢 声卡<br />🟢 内置扬声器<br />🟢 内置麦克风<br />🟢 耳机插孔 | 驱动：AppleALC，使用 [layout-id 101](https://github.com/acidanthera/AppleALC/blob/master/Resources/ALC257/Info.plist)<br /> |
| 内置键盘   |                             | 🟢 小键盘<br />🟢 背光控制<br />                             | 驱动：[VoodooPS2Controller](https://github.com/acidanthera/VoodooPS2) |
| 内置触控板 | Synaptics `SYNA0000`        | 🟢 中断模式<br />🟢 多指/手势识别<br />🟢 防误触检测          | 驱动： [VoodooRMI](https://github.com/VoodooSMBus/VoodooRMI) (I2C模式) + [VoodooI2C](https://github.com/VoodooI2C/VoodooI2C/pull/532) (使用适配AMD的v2.9) |
| USB        | XHC0<br />XHC1              | 🟡                                                          | 所有USB接口可用，包括C口，最高USB3.2 Gen1 速率<br />驱动：[GUX-RyzenXHCIFix](https://github.com/RattletraPM/GUX-RyzenXHCIFix) (AMD锐龙专用魔改) |
| 电源/电池  |                             | 🟢                                                          | 电量信息显示： [SMCBatteryManager.kext](https://github.com/acidanthera/VirtualSMC)<br /> |
| 内置屏幕   | 15.6' 1080P  144Hz          | 🟢 144Hz高刷<br />🟢 屏幕背光亮度控制                        |                                                              |

### 功能

| Item     | Status                                                       | Notes                                                        |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 睡眠     | 🟢 开合盖进入/退出睡眠<br />🟡 睡眠（Sleep）<br />🔴 深度休眠（Hibernate，写入硬盘&断电） | 现用优化措施：<br />- 引入[CpuTscSync](https://github.com/Seey6/CpuTscSync) (修改版，针对AMD移动处理器优化，改善睡死问题)<br />- 升级BIOS版本到FSCN28WW（解决各种奇怪睡眠问题，请自行操作）<br />- 关闭深度休眠（见下文介绍） |
| 联想fn键 | 🟢 完美支持（状态显示&软件控制）<br />- F1-F4&Home-PgDn 音频控制<br />- F5-F6 屏幕亮度控制<br />- F8 飞行模式<br />- F10 触控板控制<br />🟡 正常可用（可正常触发）<br />- fn+Q 模式切换 （无法软件控制，无触发状态显示）<br />- fn+Space 键盘背光控制（可软件控制，无触发状态显示）<br />- fn+Esc Fn锁（可软件控制，无触发状态显示）<br />🔴 无法使用：其他未列出的fn键 | 控制驱动&配套软件：[YogaSMC](https://github.com/zhen-zen/YogaSMC) |
| 苹果服务 | 仅列出不可用：<br />🔴 隔空投送、接力、通用控制<br />🔴 跨设备同步专注状态/屏幕使用时间<br />🔴 部分App：家庭、iMessage、FaceTime<br />🔴 随航 | 当前使用的Intel无线网卡驱动不兼容。<br />更换常见的白苹果无线网卡后可用（除了随航，需要等待显卡驱动修复硬解）<br /> |

## 已知问题

1. 打开部分应用会引起花屏/卡死/崩溃。

   > 关联：[Advanced OpenGL apps may have artefacts or freeze the system · Issue #158 · ChefKissInc/NootedRed (github.com)](https://github.com/ChefKissInc/NootedRed/issues/158)

   - 存在问题的部分应用：Chrome、Edge、Notion、Firefox

   - 问题：显卡驱动[NootedRed](https://github.com/NootInc/NootedRed)和这些应用所使用的新版OpenCL不兼容，有待显卡驱动更新修复。

   - 临时解决方案：

     - 在问题应用的设置内，关闭硬件加速

     - （驱动作者不推荐）引入[BFixup.kext](https://github.com/ChefKissInc/NootedRed/issues/158#issuecomment-1842011907)，降级显卡驱动的OpenGL

     - （推荐）使用终端启动App，并附加启动参数（禁用使用GPU合成UI）
       示例：

       ```shell
       open -a "Microsoft Edge.app" --args --disable-gpu-compositing
       ```

       > 提示：你可以将该附加了启动参数的命令，使用自动操作（Automator）打包成新的`.app`应用程序。
       
     - 对于 Firefox 浏览器，上述的`BFixup.kext`和启动参数都无效，你可以使用[旧版本Firefox 79.0](https://archive.mozilla.org/pub/firefox/releases/) (感谢 [@yaxirhuxxain](https://github.com/jimlee2048/Hackintosh-Lenovo-Legion-R7000P2020H/issues/1#issuecomment-1935373999))

2. 打开部分应用可能会引起卡顿。

   - 解决方案：使用[UMAF](https://github.com/DavidS95/Smokeless_UMAF)手动设置显存为2G

3. 视频硬解不可用

   > 关联：[Image & Video hardware decoding/encoding is dysfunctional (github.com)](https://github.com/ChefKissInc/NootedRed/issues/28)

   - 显卡驱动问题，暂时无解，等待更新修复。

4. 偶现系统启动失败，卡在某个地方。

   - 临时解决方案：长按电源键强制关机后，再尝试重新启动。不行就尝试多强制重启几次。
   - 解决方案：禁用XHC0（ACPI->勾选`SSDT-XCH0-DISABLE.aml`）& 不使用魔改XHCI驱动（Kext->取消勾选`GenericUSBXHCI`）。
     - 注意：该解决方案会禁用掉2个USB接口（顶部C口&左侧A口），请自行取舍。

5. 偶现系统启动后，键盘/触控板无法正常使用。

   - 解决方案：重启系统。

6. 睡眠唤醒后，蓝牙可能停止工作，无法连接到任何设备。

   - 唤醒后等待一会，观察1-2min后是否能重新自动连上，有时能够自动恢复。
   - 如果不行，可尝试手动开关蓝牙。
   - 如果尝试手动开关后，蓝牙仍无法正常工作，则重启系统。
   - 更换常见白苹果无线网卡后可以有效缓解该问题。

7. 长时间睡眠后，屏幕黑屏无法正常唤醒，且此时系统仍能正常工作（可以听到键盘提示声）

   > 关联：[Black screen after a long sleep, but the system works · Issue #213 · ChefKissInc/NootedRed (github.com)](https://github.com/ChefKissInc/NootedRed/issues/213)

   - 显卡驱动问题，暂时无解，请强制重启。

8. 深度睡眠（hibernate）不可用，深度睡眠重启后可能黑屏/蓝屏/花屏卡死。

   - 暂时未能找到修复方法，请关闭深度睡眠模式。

     ```shell
     sudo pmset -a hibernatemode 0
     sudo pmset -a autopoweroff 0
     sudo pmset -a standby 0
     ```


## 安装指南

🚧 WIP 施工中

可参考：

- [W2725730722/Lenovo-R7000P-2020-Hackintosh: 联想拯救者 R7000P 2020 黑苹果 (github.com)](https://github.com/W2725730722/Lenovo-R7000P-2020-Hackintosh)
- [OpenCore安装指南 (sumingyd.github.io)](https://sumingyd.github.io/OpenCore-Install-Guide/)



## 致谢

- [Apple](https://www.apple.com/) for macOS.
- [acidanthera](https://github.com/acidanthera) 开发的OpenCore和大量重要kext。
- [NootedInc](https://chefkissinc.github.io/) 开发的NootedRed、VoodooI2C对AMD的适配、以及成员[@VisualEhrmanntraut](https://github.com/VisualEhrmanntraut)的帮助。
- [W2725730722](https://github.com/W2725730722/Lenovo-R7000P-2020-Hackintosh) 珠玉在前，本仓库工作基于其提供的Sonoma EFI继续演化修改而来。
- [Dortania](https://dortania.github.io/)、[daliansky/OC-little](https://github.com/daliansky/OC-little)、[5T33Z0/OC-Little-Translated](https://github.com/5T33Z0/OC-Little-Translated) 等翔实细致的教程指南。
- 以及Hackintosh社区所有爱折腾、爱分享的朋友，祝大家玩的开心:) 
