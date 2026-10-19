完整源码见 [harmony-operation-journal-source.zip](harmony-operation-journal-source.zip)，下载解压后用 DevEco Studio 打开。

# 操作手记 · 0.3

手机独立运行的鸿蒙操作记录实验应用。**记录列表、中文时间线已实现并更新到测试手机；跨应用按钮采集和自动复现尚未实现可用。** 不应把它当作完整的全局操作录制器。

## 本次改动

- 每次“开始新记录”至“结束并保存记录”形成一条记录，列表显示时间、事件总数和跨应用事件数。
- 点开记录按发生顺序查看中文操作说明及可用截图；运行信息默认折叠，原始事件可以展开。无可用截图时明确说明。
- 旧版本的日志合并迁入一条历史记录，不丢弃原数据。跨应用未知事件不会被替换为本应用的坐标或虚构按钮。
- 支持导出本次或全部事件。JSON/CSV 暂不打包图片。
- 自动复现明确显示不可用及原因，没有用截图播放替代自动操作，也没有执行点击的后台任务。

## 目前的真机结果

0.3 已编译、签名并通过 USB 保留数据升级；列表、旧数据迁移和详情页已检查。旧版界面显示本页截图测试成功；这不证明跨应用事件截图正常。目前观察到的辅助服务连接信号和跨应用事件均为零。手机无需连接电脑即可打开应用，但核心的全局记录与复现需求尚未满足。

公开 SDK 的旧无障碍回调、控件查询与执行动作接口已经废弃。官方文档说明相关能力不再开放；不能根据编译通过推定当前手机可执行跨应用操作。保留的 retrieve 扩展仅用于兼容性验证，必须由使用者在系统设置中手动启用（若系统提供）。没有手势注入、按键观察或自动授权。需要厂商提供受支持的接口或集成途径，才能继续承诺跨应用自动复现。

## 采集范围与限制

- 实际收到的亮灭屏、锁定解锁、充电及手电筒状态。系统状态不能证明谁点击了哪个按钮；手电筒监听初始通知也不是开关操作证据。
- 可选的 CUSTOM_SCREEN_CAPTURE 测试及旧辅助服务事件验证。事件包含系统实际提供的应用包名、非输入控件标签、组件区域和时间。包名未知时明确标记；只对明确的系统包名显示中文名称。
- 组件区域不是真实触点；界面变化不等于新应用启动。事件后的截图可能已是操作后的画面。
- 输入控件、密码控件或属性未知时跳过文字和截图。截图可能包含屏幕上的其他私人内容。
- 最多20,000条事件和100张截图，早期数据会过期，空记录仍保留在列表。排队上限12，超出会记录缺口。进程挂起或系统不送达会漏记。
- 结束后停止写入；进程重启默认暂停并结束之前的记录。清空日志不会立即删除截图文件。没有全天候后台保活。
- 0.3 删除了首页全屏坐标监听和测试按钮，避免把界面自身点击当作用户期望的全局操作；旧数据仍可查看。

## 构建与验证

DevEco Studio 26.0.0.851 / HarmonyOS SDK 26.0.0.105 / Hvigor OHOS Plugin 6.26.8。工程 API 12 基线；新截图接口需要更高 API。DevEco 打开后配置自己的自动签名，并确认目标 product 的 signingConfig 指向生成配置。

```sh
python3 -m unittest discover -s tests -v
```

13项检查覆盖实际SQL的暂停门禁、迟到写入、容量、绑定参数、导出顺序、记录分组、空记录隔离、迁移、重启结束以及资源引用。SQLite检查和编译均不能证明跨应用能力。详见 [真机结果](DEVICE_TESTS.md) 和 [编译结果](BUILD_VERIFICATION.md)。

## 官方依据

- [OpenHarmony 官方辅助功能上下文文档](https://github.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-accessibility-kit/js-apis-inner-application-accessibilityExtensionContext.md)
- [华为无障碍扩展接口](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-application-accessibilityextensionability)
- [官方接口声明](https://github.com/openharmony/interface_sdk-js/blob/master/api/@ohos.application.AccessibilityExtensionAbility.d.ts)
- [官方截图接口](https://github.com/openharmony/interface_sdk-js/blob/master/api/@ohos.screenshot.d.ts)

MIT。公开仓库存放完整源码ZIP和说明，不包含个人截图、操作日志、设备标识、签名证书或口令。详见 [隐私说明](PRIVACY.md)。
