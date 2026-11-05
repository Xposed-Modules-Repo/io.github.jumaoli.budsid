# BudsId · 小米耳机代号替换

一个 LSPosed 模块：把小米蓝牙识别到的耳机代号（deviceId）换成另一款小米耳机，从而直接复用目标型号的**连接弹窗动画**、**超级岛通知**和**设置页设备图**。只改“显示身份”，不动 A2DP/HFP 音频链路。

- 作用域：`com.xiaomi.bluetooth`（弹窗 / 超级岛 / 设备详情）、`com.milink.service`（融合设备卡）
- 适配：HyperOS 2/3/4（Android 17 实测）
- 源码：https://github.com/jumaoli/budsid-lsp

## 原理

小米蓝牙的整套视觉资源都按耳机代号取：

| 来源 | 代码 |
| --- | --- |
| 广播解析 | `MiuiAdvDataInfo(V2)#getDeviceId()`、`MiuiHeadsetInfo(V2)#getDeviceId()` |
| 缓存 | `DeviceIdCached` 里的 `saved<MAC>` |

代号由广播字节拼成（`String.format("%06X%04X", majorId, minorId)`）。模块在 `getDeviceId()` 返回处替换字符串，并同步替换缓存读取，下游 `DeviceClassify`、`MiuiCloudParser`、`MiuiBluetoothNotification` 就会自然加载目标型号资源：

```
弹窗逐帧   .../fc_resources/<deviceId>/fc_<deviceId>_<frame>.webp
资源描述   .../fc_resources/<deviceId>/resource_record.xml
超级岛     .../files/my_internal_files/target/<deviceId>/*
```

同时兜底 hook `MiuiBluetoothNotification#getResourceUri` 与 provider 查询，避免任何界面漏出真实代号。全程只读，不改系统文件。

## 使用

1. 安装 APK，LSPosed 启用模块，勾选 `com.xiaomi.bluetooth`、`com.milink.service`；
2. 打开 App，选目标型号（可搜索代号库），点「确定」；
3. 重新连接耳机，弹窗 / 超级岛 / 设置详情页即换成目标型号；
4. 弹窗、超级岛、详情页三个开关可分别控制。

代号库来自小米 CDN（`00000000.zip` → `resource_record.xml`，约 455 条，含 10 位外设代号），每天自动更新；本地列表补充云端缺失的型号（如小米 AI 眼镜 `01013304`）。

## 界面

对齐 SukiSU Ultra 的 Miuix 风格：GPU `RenderNode` 4dp 实时模糊、AGSL 液态玻璃底栏（折射 + 重力跟随高光）、可拖动带过冲的悬浮导航气泡。

## 注意事项

- 仅替换显示身份，音质、ANC、固件行为仍是真实型号；
- 弹窗 / 超级岛资源由蓝牙进程按新代号下载，首次连接可能先显示占位图，重连一次即可。
