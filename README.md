> 完整源码以 [harmony-operation-journal-source.zip](harmony-operation-journal-source.zip) 提供。下载解压后用 DevEco Studio 打开。源码已编译成功，但未完成签名和真机测试；未签名 HAP 不能直接安装。

# 操作手记 · HarmonyOS Operation Journal

用于个人手机的透明、本地事件记录实验应用。用户目标设备：Mate 80 Pro Max / 鸿蒙 7（用户提供）；工程基线：HarmonyOS 5.0 / API 12，ArkTS + ArkUI。

**状态：源码原型，已在 DevEco Studio 26.0.0.851 / HarmonyOS SDK 26.0.0.105 编译成功，生成未签名 HAP；尚未签名、安装或在鸿蒙 7 真机验证。不是“全手机全部操作记录器”。未签名 HAP 不能直接安装。**

构建环境和限制见 [编译验证记录](BUILD_VERIFICATION.md)。

## 已编写的功能

- 用户主动开始/暂停，进程重启后默认暂停。
- 应用内部触点抬起的屏幕坐标（px）和测试按钮操作。
- 动态订阅亮屏、灭屏、锁屏、解锁、接电、断电公共事件；实际送达取决于设备、权限及运行状态。
- 监听 CameraManager 的手电筒状态变化，不打开相机、不主动开灯。
- 本地 SQLite 日志、时间线筛选、JSON / CSV 文件选择器导出、清空确认。
- 最多保留最近 20,000 条；界面展示最近 200 条。
- 记录暂停与事件写入在同一数据库中判断，已暂停后到达的异步回调不能绕过写入开关。

## 做不到的范围

| 需求 | 本版实际覆盖 |
| --- | --- |
| 全手机原始点击坐标 | 不支持；只采集本应用内部触点 |
| 其他应用点击哪个按钮 | 未接入；不能以节点边界或节点中心冒充真实点击坐标 |
| 任意应用内业务操作 | 无法推断；例如点击开关不代表操作已经成功 |
| 手电筒开关 | 仅保存系统实际回调；不等于拍照闪光灯或智能家居灯光 |
| 屏幕亮灭 | 动态公共事件；无回调不会补造 |
| 后台全天候无遗漏 | 不支持；系统挂起、杀进程、重启、权限变化均可能导致空档 |

官方无障碍文档明确说明，旧 `onConnect` / `onAccessibilityEvent` 回调从 API 12 废弃。OpenHarmony 当前官方文档写明系统不再开放相关能力；新接口在 SDK 类型声明中标注为系统接口。**不能据此宣称普通鸿蒙 7 应用可以全局监听。**
`experimental/LegacyAccessibilityAbility.ets` 仅是未启用、未纳入构建的历史接口适配参考，不是绕过系统限制的方法。正式模块没有注册无障碍扩展，没有系统签名权限。

## 在 DevEco Studio 运行

1. 安装华为官方 DevEco Studio 及支持 API 12 的 HarmonyOS SDK；工程 modelVersion 为 5.0.0，已使用 DevEco Studio 26.0.0.851 自带的 Hvigor 6.26.8 编译。当前 IDE 若要求升级，使用 IDE 的工程升级并按已安装 SDK 调整 `build-profile.json5` 中版本。
2. 直接打开仓库根目录，同步 Hvigor / OHPM 依赖。仓库没有附带下载型 wrapper 或签名材料；使用 IDE 内置工具。
3. 配置你自己的 bundleName 和开发者签名，在 **Project Structure → Signing Configs** 使用自动签名，选择已连接的手机。不要提交 IDE 生成的证书、profile、私钥或含本机路径的配置。
4. 连接已开启开发者模式和调试的测试手机，运行 `entry`。不需要网络权限。
5. 点击“开始记录”，点击测试按钮，检查应用内坐标和按钮事件；下拉控制中心切换手电筒、锁屏再亮屏，返回核对实际日志与注册状态。收不到的事件应作为兼容性缺口记录。
6. 暂停后继续操作，核对不增加日志；导出 JSON/CSV，检查时间、来源、数量；清空后重启确认为空且暂停。

开发工具和 SDK 已安装，ArkTS 编译已通过；以上手机操作 **尚待签名及真机执行**。后台保活没有伪装成音频或定位任务，没有加入无限后台服务。

## 验证

```sh
python3 -m unittest discover -s tests -v
```

测试从 ArkTS 源码提取 SQL，在 SQLite 中验证暂停门禁、迟到事件、保留上限、排序/导出集合以及数据库重开，并验证资源和 manifest 引用。它们**不替代 ArkTS 编译、API 类型检查或设备兼容性测试**。

见 [验收清单](DEVICE_TESTS.md) 和 [隐私说明](PRIVACY.md)。

## 官方接口依据

- [华为手电筒状态监听](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/camera-torch-use-V5)
- [华为 CommonEvent 事件列表](https://developer.huawei.com/consumer/en/doc/harmonyos-references-V14/capi-common-event-V14)
- [华为 AccessibilityExtensionAbility](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-application-accessibilityextensionability)
- [OpenHarmony 无障碍文档（官方源码）](https://github.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-accessibility-kit/js-apis-application-accessibilityExtensionAbility.md)
- [OpenHarmony 接口声明（官方源码）](https://github.com/openharmony/interface_sdk-js/blob/master/api/@ohos.application.AccessibilityExtensionAbility.d.ts)

## 许可

MIT。公开仓库存放源码，不存放手机日志、证书和任何个人操作数据。
