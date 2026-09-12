# HiClock Studio

HiClock Studio 是 Windows WPF 管理程序（.NET 8），工程完全位于 `HiClock/HiClock Studio` 内。

> 本文档为 Studio 唯一说明文档，整合了原 `README.md`、`ARCHITECTURE.md`、
> `docs/DEVICE_PROTOCOL.md`、`docs/SCREEN_STREAM_FLOW.md` 与 `tools/README.md`
> 的内容。技术细节与最新状态以本文档及源码为准。

---

## 1. 当前已完成

- `.NET 8 + WPF` 多项目解决方案。
- 读取 `.hcb` 运行时单文件样式字库包、`.hcs.h` 可编译样式包，以及旧版 `.h` 设备字库样式（打开时自动匹配系统字体）；撤销快照与 `.hcs.h` 解析仍通过 `LoadJson`/`LoadHeader` 保留兼容。
- 保存/导出仅保留两种格式：`.hcb` 运行时单文件样式字库包（上传即用，免编译）与 `.hcs.h` 可编译单文件格式（内嵌样式 + 编译期字库，include 后编译烧录）。
- 对未知 JSON 字段进行保留，降低打开旧格式样式造成数据丢失的风险。
- TFT 画布预览、元素命中选择和拖动。
- 样式列表、图层列表和基础属性编辑。
- 100 步撤销/重做，连续输入自动合并为一次历史操作。
- 文本、矩形、圆形和直线元素创建，以及复制、删除、图层移动和居中。
- 50%～300% 画布缩放。
- 读取 `HiDesign/fonts` 的 LVGL 4 BPP 位图字体并逐像素预览。
- Photoshop 风格紧凑属性栏，按文字、矩形、直线和圆形动态显示相关属性。
- 矩形四角独立圆角，以及日期/信息元素的日期格式与公历/农历设置。
- 画布和图层列表支持 Ctrl/Shift 多选、框选、整组拖动、复制和删除。
- 拖动时显示对齐辅助线，松手自动吸附；支持边缘对齐和水平/垂直均匀分布。
- 矩形和圆形拖拽缩放手柄，以及图层置顶、置底、上移和下移。
- 读取并写回按屏幕尺寸分组的固件 `.hcs.h` 样式包。
- `char_colors` 和 `custom_font` 强类型读写，逐字符颜色参与位图字体预览。
- 系统颜色选择器，以及自定义文字逐字符设色和恢复默认色。
- 样式按画布尺寸筛选、多选、全选、选择导出、清空和拖拽排序。
- 图层列表顶部对应最高图层，支持多选和拖拽调整层级。
- 多选属性批量编辑：整体位置、颜色、显示状态、字体、对齐以及各图形专属属性。
- 字体目录可切换并即时刷新普通下拉框，字体目录、最近样式目录和新建画布尺寸自动记忆。
- 文本元素可直接使用 Windows 已安装字体进行设计预览，并单独设置像素字号。
- “设备字体包”导出按样式使用的字符将 Windows 字体栅格化为 LVGL 4 BPP 数据，输出 `.hcb` 运行时单文件样式字库包（新版固件直接接收，运行时解析并合并重复字形），或 `.hcs.h` 可编译单文件格式（含样式与编译期字库，免上传直接编译烧录）。
- 每个样式拥有独立的 100 步撤销/重做历史，切换样式后不会丢失。
- 动态文本 `binding/prefix/suffix/decimals/fallback` 模型。
- 手动连接设备和并发扫描本机活动 IPv4 `/24` 网段；后续支持 mDNS（`hiclock-XXXX.local`）并行探测，PC 重启换 IP 后秒级恢复连接。
- 兼容当前固件接口的样式上传与启用（上传统一走 `.hcb` 单文件包）。
- 已实现对新版固件的 `/api/v1/device` 能力协商与 `/api/v1/telemetry` 遥测推送（旧版 `ApiVersion=0` 设备仅支持样式管理）。
- Windows CPU、内存、前台进程和前台窗口标题采集。
- LibreHardwareMonitor 动态适配：检测到 DLL 后读取 CPU/GPU、主板、存储、网络、温度、负载、功耗、显存和风扇等传感器。
- PawnIO 普通权限硬件后端：自动检测已安装的 PawnIO；不可用时保留厂商 API 和 Windows 指标并明确降级。
- PresentMon 动态适配：按前台 PID 管理采集进程并计算最近窗口 FPS/帧时间。
- 多设备遥测发送限流：同一设备上一请求未完成时丢弃旧快照。
- 左侧主导航分离“设备管理”和“样式编辑器”，设备页支持单台/批量打开后台。
- USB 设备连接稳定性：`UsbDeviceBindingService` 持有 USB CDC 端口生命周期，针对 arduino-esp32 #8651（Windows `usbser.sys` 在 `Open()` 断言 DTR/RTS 被 ESP32-C3 ROM 解读为硬复位）以 `SerialPort.Handshake=RequestToSend` + 设备级复位抑制 + 端口新鲜度门禁规避端点楔死；`WaitForPortStableAsync` + `PurgeAbsentBindings` 清理残影；首开即时绑定把首连压到 ~1–2s。
- 每台设备可独立开启“推送串流数据”，进入串流模式并在取消后恢复原显示模式。
- 屏幕/窗口/媒体/数据四种 USB 串流来源（详见第 8 章）。
- 本机 AI 样式 API：Studio 运行时监听 `http://127.0.0.1:17340`，AI/自动化工具可读取、新增或替换样式，修改会立即同步到编辑画布。接口说明见 `/api/v1/guide`，OpenAPI 文档见 `/openapi.json`；服务仅绑定回环地址且不开放 CORS。
- Ring0Service：独立的 Windows `SYSTEM` 服务，以最高权限采集普通进程无法读取的内核态数据，经命名管道向主程序推送（详见第 6 章）。
- 单实例保护：同一会话内只允许一个 HiClock Studio 进程，重复启动会激活已有窗口后直接退出，避免多实例争抢设备串口与 AI API 端口；启动脚本不再强杀运行中的实例。

常用快捷键：

```text
Ctrl+Z       撤销
Ctrl+Y       重做
Ctrl+D       复制当前元素
Delete       删除当前元素
```

---

## 2. 解决方案结构

```text
HiClockStudio.sln
└── src/
    ├── HiClockStudio.Core          样式模型、JSON 兼容、遥测模型（不依赖 WPF/网络/Windows API）
    ├── HiClockStudio.Device        扫描、设备注册、HTTP 协议（HiClockClient）
    ├── HiClockStudio.Monitor       Windows 状态采集（产生 TelemetrySnapshot）
    ├── HiClockStudio.App           WPF UI、画布和编排
    └── HiClockStudio.Ring0Service  Windows SYSTEM 服务（独立进程，命名管道向 Monitor 推送内核态数据）
```

依赖方向：`App → Core / Device → Core / Monitor → Core`；`Ring0Service` 为独立 SYSTEM 服务，
主程序（普通 `asInvoker` 权限）经 `Monitor.Ring0ServiceClient` 通过命名管道取数，不直接触碰内核驱动。

`Core` 不引用 WPF、网络或 Windows API，因此样式模型可以单独测试。`Device` 不引用 WPF，
可在后续复用于命令行部署工具。`Monitor` 只负责产生 `TelemetrySnapshot`，不直接连接设备。

---

## 3. 样式兼容策略

每个样式和元素同时保存：

1. 编辑器认识的强类型属性。
2. 原始 `JsonObject` 的深拷贝。

序列化时在原始对象上覆盖已编辑字段。`char_colors`、`custom_font`、`dateCalendar`、
`dateFormat` 和矩形四角圆角均已进入强类型模型；未识别的新字段仍通过原始对象深拷贝保留，
不会因为打开并重新保存而消失。

WPF 可保存两种格式：`.hcb`（含样式与运行时字库的单文件包，上传即用）或 `.hcs.h`（可编译单文件格式，含样式与编译期字库，include 后编译烧录）。固件已移除 JSON 直传接口
`/api/clock/styles/import`，统一走 `.hcb`。文本元素额外保存 `font_source`、`font_family` 和
`font_size_px`：`windows` 模式只用于设计器预览，设备包导出时会转换成固件可识别的
LVGL 4 BPP 字库 ID。

画布渲染器从用户选择的字体目录按需读取 LVGL 4 BPP `.h` 字体，默认目录是程序输出目录的
`fonts`。旧样式仍按设备字库模式打开；Windows 字体模式通过 WPF 使用系统已安装字体进行
设计预览。点击“设备字体包”时，导出器收集样式实际使用的字符，并在独立 STA 线程中将
TTF/OTF 字形栅格化为 LVGL 4 BPP 数据，再按目标格式合并：`.hcb` 将样式 JSON 与运行时 `.bin`
打包；`.hcs.h` 将样式 JSON 与编译期字库数组生成单文件头。

`.hcb` 使用固定头部、JSON 清单和二进制 payload：固件先写临时文件并验证，再把样式存入
`/styles/`、字库存入 `/fonts/`。同一字体族/字号在多个样式间共享一个资源文件；后续上传会按
Unicode 合并新字形。删除样式时只删除样式 JSON；只有 Flash 进入低空间水位时，删除接口才会
扫描全部剩余样式的 `font_glyphs` 引用，并重写/删除未引用的字形资源。

---

## 4. 设备兼容策略

探测顺序：

1. `GET /api/v1/device`：新版设备能力协商。
2. `GET /api/clock/styles`：当前固件兼容识别。

旧版设备 `ApiVersion=0`，只允许样式管理，不发送遥测。新版设备声明 API 版本后才启用遥测推送。

局域网扫描采用活动网卡 `/24` 地址段并发探测，最大并发 32。设备以 eFuse MAC 高 16 位注册
`mDNS` 主机名 `hiclock-XXXX.local`；`HiClockDevice.MdnsHostName` 推导同主机名，
`ProbeKnownDevicesAsync` 将「已知 IP 探测」与「mDNS 主机名探测」并行（`Task.WhenAll`），
PC 重启换 IP 后秒级恢复连接，绕开扫描整个 `/24`（254 地址）的极慢路径。

---

## 5. 遥测时序

Windows 基础指标每 500ms 采样一次。每台设备最多只有一个进行中的遥测请求；如果设备响应速度低于
采样频率，新快照不会排队。这样可以保证 TFT 收到的是最新状态，并避免网络恢复后播放积压数据。

```text
采样 → TelemetrySnapshot → 检查设备能力 → 最新值覆盖 → POST /api/v1/telemetry
```

---

## 6. 权限

主程序清单使用 `asInvoker`，不申请管理员权限。CPU、内存、前台应用和显卡厂商 API
可在普通用户权限运行；已安装 PawnIO 时，低层硬件访问由其签名驱动完成。PawnIO 不可用
或 PresentMon 无权创建 ETW 会话时显示降级提示，避免整个设计器长期高权限运行。

**Ring0Service（内核态采集）**：普通权限主程序无法读取的部分内核态数据，由独立
`SYSTEM` 服务 `HiClockRing0` 采集。该服务加载 Ring0 内核读写能力（如 LibreHardwareMonitor
的 Ring0 驱动），经命名管道（`PipeServer("HiClockRing0")`）向 `Monitor.Ring0ServiceClient`
推送最新 JSON 快照；`OnStart` 启动 500ms 周期采集循环，`OnStop` 取消；驱动未就绪时限频 5s
重试。支持 `--console` 以普通进程调试。**修改 Ring0 代码后必须重新发布 dll/exe 并重启
`HiClockRing0` 服务**，否则主程序经管道仍读旧代码。安装随 `编译正式版本.bat` 一并 publish 到
`tools/Ring0Service`。

---

## 7. 设备协议 v1（草案）

本文件描述 HiClock Studio 与固件之间的 v1 接口。串流画面完全由 Studio 渲染，
设备只接收 HCSF 像素帧；RAM 样式预览仍是预留协议。

### 7.1 设备能力

```http
GET /api/v1/device
```

```json
{
  "id": "A1B2C3D4E5F6",
  "name": "桌面时钟",
  "firmware": "1.2.0",
  "api": 1,
  "screen": {
    "width": 240,
    "height": 320,
    "rotation": 0
  },
  "features": [
    "style-preview",
    "telemetry",
    "stream-mode"
  ],
  "streamRenderer": "host",
  "streamTransport": {
    "protocol": 1,
    "usb": true,
    "udp": false,
    "formats": ["jpeg", "rgb565", "tile565"]
  }
}
```

`id` 必须稳定且不随 DHCP 地址变化。建议使用完整芯片 MAC 派生值，不使用当前四位显示后缀作为内部 ID。

### 7.2 本机遥测

遥测数据不再通过 HTTP 发给设备。Studio 在本机采样后直接代入本地 `stream` 样式，
把最终画面编码为 RGB565 关键帧或 16×16 增量 Tile。设备不存在
`POST /api/v1/telemetry` 接口，也不保存 `streamBindings`。

Studio 当前可发送的性能绑定包括：

```text
cpu.usage  cpu.temp  cpu.temp.max  cpu.clock  cpu.power  cpu.voltage
gpu.usage  gpu.temp  gpu.temp.hotspot
gpu.clock  gpu.memory.clock  gpu.power  gpu.vram.used  gpu.vram.total
gpu.fan.rpm  gpu.fan.percent
ram.usage  fan.rpm  fan.cpu.rpm
storage.temp  storage.usage
network.download  network.upload
app.name  app.fps  app.frame_ms
```

显存容量单位为 GiB，网络速度单位为 Mbit/s，其余单位由绑定元素的 `suffix` 决定。
具体指标是否存在取决于硬件、厂商驱动以及 PawnIO/LibreHardwareMonitor 对该设备的支持。

### 7.3 串流显示模式

```http
POST /api/v1/stream/mode
Content-Type: application/json
```

```json
{ "enabled": true, "renderer": "studio" }
```

开启前 Studio 从专用串流目录读取设备【设置】中选定且尺寸匹配的已保存样式，并把
设备可选择互斥的数据模式或显示器模式；前者只渲染保存后的串流样式和实时遥测数据，后者只渲染选定的 Windows 显示器。两种选择分别保存且不会叠加，编辑器未保存内容不会进入运行快照。
设备写入一次性独占票据并软重启，独占模式只初始化 USB、HCSF 接收器、帧缓冲和 TFT；
Studio 使用 `HICLOCK_STOP` 退出。未连接 USB 时 Studio 不允许开启串流。`stream` 样式 JSON
仅属于 Studio 本地工程，不写入设备 Flash/SD。

### 7.4 RAM 样式预览

```http
POST /api/v1/styles/preview
Content-Type: application/json
```

```json
{
  "requestId": 42,
  "style": {}
}
```

该接口只能更新临时 `ClockStyleConfig`，不得写 LittleFS 或 Preferences。

```http
DELETE /api/v1/styles/preview
```

退出预览并恢复原正式样式。

### 7.5 正式样式

当前桌面程序上传正式样式统一使用推荐的单文件样式字库包。Studio 以 multipart 文件上传 `.hcb`，固件会先
校验并解包，再提交样式和运行时字库；上传成功后通过 `setclockstyle` 启用：

```text
POST /api/clock/styles/bundle
GET  /setclockstyle?style={id}
```

（旧 JSON 直传接口 `/api/clock/styles/import` 已移除，不再支持。）

bundle 内的样式 JSON 可以包含 `font_glyphs` 对象，用于低空间时的全量引用扫描。正常删除样式
不会立刻删除字形；只有删除动作发生在存储空间低于水位时，固件才执行一次字形级垃圾清理。

未来可增加：

```text
POST /api/v1/styles
POST /api/v1/styles/{id}/activate
```

### 7.6 动态文本

```json
{
  "text": {
    "enabled": true,
    "x": 10,
    "y": 20,
    "layer": 1,
    "font": "MiSans_Bold_28px",
    "color": "#ffffff",
    "align": "left",
    "binding": "cpu.usage",
    "prefix": "CPU ",
    "suffix": "%",
    "decimals": 0,
    "fallback": "--"
  }
}
```

旧固件会把它作为普通 `text` 元素处理并忽略新增字段；实现 v1 动态渲染后，存在 `binding` 时显示遥测值。

---

## 8. 屏幕串流链路详解（PC 屏幕 / 窗口 / 媒体 / 数据 → TFT）

> 适用范围：全部 `stream` 样式。Studio（Windows）采集本机遥测和可选屏幕源，
> 完整渲染最终画布 → 编码分片 → USB CDC → ESP32-C3 固件校验组装 → TFT 推屏。
> 协议字段与设备能力见第 7 章「设备协议 v1」。

### 8.1 总体架构

```
┌─ HiClock Studio（Windows）────────────────────┐        ┌─ ESP32-C3 固件 ──────────────────────┐
│ TelemetryCollector / ScreenCaptureService     │        │ screen_stream.h  USB CDC 收包          │
│ StreamCanvasRenderer  完整画布合成             │──USB──▶│ 魔数重同步 → CRC/序号校验 → 整帧组装   │
│ ScreenStreamService   RLE/JPEG/Tile 分片        │        │ RLE/JPEG解码/Tile更新 → 分段整帧缓冲     │
│ UsbDeviceBindingService 1KB低水位分块写入      │        │ 解码/2×放大 → 32行分段帧缓冲             │
└───────────────────────────────────────────────┘        │ VSync 消隐 → DMA pushImage → ST7789   │
                                                          └───────────────────────────────────────┘
```

关键代码文件：

| 端 | 文件 | 职责 |
|---|---|---|
| Studio | `HiClock Studio/src/HiClockStudio.App/Services/ScreenCaptureService.cs` | 屏幕采集（GDI StretchBlt，懒启动，最新帧覆盖） |
| Studio | `%LocalAppData%/HiClock Studio/StreamStyles` | 点击【保存】发布的运行时串流样式库，按设备尺寸筛选 |
| Studio | `HiClock Studio/default-stream-styles.json` | 可由编辑器打开修改、构建时嵌入软件的默认串流样式源 |
| Studio | `HiClock Studio/src/HiClockStudio.App/Services/StreamCanvasRenderer.cs` | 复用编辑器画布，按设备设置中的互斥模式渲染已保存样式或显示器最终帧 |
| Studio | `HiClock Studio/src/HiClockStudio.App/Services/ScreenStreamService.cs` | JPEG/RGB565/Tile 编码、HCSF 分片打包、发送循环 |
| Studio | `HiClock Studio/src/HiClockStudio.App/Services/UsbDeviceBindingService.cs` | COM 口探测绑定、写队列限流、STOP/LOG 控制命令 |
| 固件 | `screen_stream.h` | 收包解析、重同步、帧组装、解码、推屏（全链路核心） |
| 固件 | `HiClock.ino` | 独占模式进入/退出、setup/loop 调度、waitForVSync |

### 8.2 四种传输形态

| 形态 | 触发条件 | 数据通道 | 渲染方式 |
|---|---|---|---|
| **显示器模式** | 设备设置选择【显示器】及 Windows 显示器 | USB CDC JPEG | 解码进整帧缓冲后推送 TFT |
| **窗口模式** | 设备设置选择【窗口】及打开的电脑窗口 | USB CDC JPEG | 解码进整帧缓冲后推送 TFT |
| **媒体模式** | 设备设置选择【媒体】及图片/视频文件或 http(s) 串流地址 | USB CDC JPEG | 解码进整帧缓冲后推送 TFT |
| **数据模式** | 设备设置选择【数据】及已保存串流样式 | USB RGB565 关键帧/Tile | 更新整帧缓冲后推送 TFT |

- 显示器/窗口/媒体与数据互斥，都由 Studio 完整渲染并只通过 USB 独占模式传输；未连接 USB 时不能开启串流。
  固件不读取 `ClockStyleConfig`，只接收 Studio 渲染完成的帧。

媒体模式（MediaStreamService + MediaUrlResolver，纯 WPF 内置能力，无新增依赖）：
- 本地/URL 图片（PNG/JPG/BMP/GIF 动图/TIFF/ICO/WebP，WebP 需系统编解码扩展），动图按帧间隔逐帧播放；
- 本地/URL 视频（Windows Media Foundation 支持的 MP4/WMV 等），`System.Windows.Media.MediaPlayer` +
  `DrawingContext.DrawVideo` 每帧渲染到 RenderTargetBitmap，经 media:// 绑定投递到屏幕源管线；
- 网页视频链接自动解析（MediaUrlResolver）：直链扩展名识别 → B 站（公开 playurl 接口）/ YouTube
  （yt-dlp 直链 → 页面 JSON → innertube 播放器 API + signatureCipher 解开）→ 通用 og:video /
  <video> / JSON 字段提取 → HEAD 探测裸流；解析失败会在设置里提示；
  YouTube 首选本地 `tools/yt-dlp.exe` 或 PATH 上的 `yt-dlp`（详见第 9 章），
  未安装时回退到内置纯 C# 解析器（尽力而为）；
- 本地路径支持 %环境变量%（如 %LOCALAPPDATA% 指向的隐藏目录）并自动归一化；文件选择器
  强制显示隐藏文件夹/文件（IFileOpenDialog FOS_FORCESHOWHIDDEN）；
- UNC/网络共享视频（\\nas\... 路径）MediaPlayer 打开失败时自动复制到本地临时缓存再播放
  （MF 对网络路径常报“找不到媒体文件”）；复制失败会提示网络路径不可访问；
- 视频循环播放、未请求时自动暂停；Live2D 原生 .moc3 需要 Cubism 运行时（未内置），
  Live2D 导出为视频/GIF 的资产可直接播放。
- 媒体音频在 Studio 本机播放（设置里可调音量 0-100，默认 80）：只走扬声器，不编码进
  USB 帧，设备端无音频通道不受影响；播放期间与画面同步，未预览/未推流时自动暂停。
- 播放控制：手动暂停/恢复、进度条 seek（墙钟累加进度）、连播/循环；单实例
  `MediaStreamService.Shared` 跨设备绑定共享。所有方法仅在 UI 线程调用（`MediaPlayer`
  与图片解码依赖 WPF `Dispatcher`）。

窗口模式（参考 OBS 窗口捕捉）：

- 顶层窗口按 `window://进程名:标题` 枚举，最小化窗口也可选择；进程名+标题在 Studio
  重启后仍可重新解析，因此窗口选择与显示器选择一样随设备设置持久化。
- 采集优先 `PrintWindow(PW_RENDERFULLCONTENT)`（可捕获被遮挡内容）；GPU/硬件加速渲染
  产生黑帧时回退屏幕可见区域 BitBlt。
- 窗口最小化后仍实时出画面：采集器在流式期间把窗口一次性移到屏幕外保持可见
  （SW_SHOWNOACTIVATE 不抢焦点、用户不可见），应用持续出帧，每帧直接 PrintWindow
  拿到实时内容；停止流式或窗口被手动还原后自动恢复窗口原位置/最小化状态。

### 8.3 HCSF 线协议

所有帧统一打包为「32 字节头 + payload」，整帧按 **1KB 固定分片**（最后一包允许不足）。

#### 8.3.1 包头（32 字节，小端）

| 偏移 | 长度 | 字段 | 含义 |
|---|---|---|---|
| 0-3 | 4B | 魔数 | `'H' 'C' 'S' 'F'`，字节流重同步锚点 |
| 4 | 1B | version | 恒为 1 |
| 5 | 1B | type | 见下表 |
| 6-7 | 2B | flags | bit0 保留；**bit1 (0x02) = 最后一分片**（`FlagLastBlock`） |
| 8-11 | 4B | session | Studio 启动时随机，区分重启前后 |
| 12-15 | 4B | frameId | 全局递增，用于丢弃乱序/过期帧 |
| 16-27 | 12B | 字段区 | 按 type 展开（见 8.3.3） |
| 28-31 | 4B | CRC32 | **payload 的** CRC32（多项式 0xEDB88320，初始/异或 0xFFFFFFFF） |

#### 8.3.2 type 一览

| type | 名称 | 当前状态 |
|---|---|---|
| 2 | JPEG 帧分片（原生尺寸完整 JPEG） | **Studio 主路径** |
| 3 | 旧 RGB565 整帧分片 | 固件保留兼容，Studio 不再发 |
| 4 | 裸流头（Direct Raw，帧后跟无头连续像素） | 旧版 Studio 路径，固件兼容 |
| 5 | 增量 Tile 帧（16×16 变化块） | 固件已实现（含 NeedsKeyframe），**Studio 当前未调用**（`BuildDeltaWireFrame` 保留） |
| 6 | 带 CRC 的 RGB565 整帧分片 | **Studio RGB565 兜底路径** |
| 7 | RGB565 RLE 无损关键帧 | **Studio 数据模式关键帧路径** |

#### 8.3.3 字段区展开（offset 16 起，全部 u16）

type=2（JPEG）：
`width(16) height(18) totalLen(20) fragIdx(22) payloadLen(24) fragCount(26)`

type=6（RGB565）：
`srcWidth(16) srcHeight(18) fragIdx(20) fragCount(22) payloadLen(24) 保留(26)=0`

type=5（Tile）：
`x(16) y(18) w(20) h(22) payloadLen(24) expectedTiles(26)`

type=4（裸流头）：
`outWidth(16) outHeight(18) bytes(20) srcWidth(24) srcHeight(26)`（兼容旧版 Studio：src 字段为 0 时可从 out 推导 2× 源）

#### 8.3.4 控制命令（ASCII 文本行，与二进制帧共用 CDC）

| 命令 | 方向 | 作用 |
|---|---|---|
| `HICLOCK_PROBE` | Studio→固件 | 探测；固件回 `HICLOCK_USB:{"deviceId","width","height","protocol":1}` |
| `HICLOCK_STOP` | Studio→固件 | 独占模式下请求退出 → 固件 ESP.restart() |
| `HICLOCK_LOG_ON` / `HICLOCK_LOG_OFF` | Studio→固件 | 打开/关闭帧率日志（fps/JPEG大小/收包/解码/推屏耗时） |

`HICLOCK_STOP` 与日志开关另有独立的原始字节旁路扫描，不依赖当前二进制帧边界；即使最后一帧残缺、主解析器仍在等待payload，也能停止独占模式或重新开启日志。Studio停止时在30ms内重复发送三次幂等STOP。

### 8.4 Studio 发送端实现细节

#### 8.4.1 采集（ScreenCaptureService）

- 后台 `PeriodicTimer(33ms)` ≈ 30 FPS；**按需启动**：只有元素被引用（`Request(binding)`）才采集。
- 采集路径三级降级：monitor DC `StretchBlt` → 虚拟桌面 DC → `CopyFromScreen`（解决副屏负坐标/混合 DPI 黑帧）。
- 缩放到最长边 ≤480（`PreviewMaxDimension`），存 BGRA + 自增 `Sequence`。
- **绝不排队**：新帧直接覆盖旧帧，避免延迟随电脑负载累积。
- 编辑器预览节流：UI 通知最多 15 FPS，防止串流开启时 WPF 队列淹没 UI 线程。

#### 8.4.2 发送循环（ScreenStreamService.RunAsync）

每 20ms 一轮（真实发送节奏由设备收包/解码/推屏与同步写回压决定）：
1. 过滤设备：USB 已连接 + 串流开启 + 已生成 Studio 本地运行快照 → 按样式 JSON 和旋转分组。
2. 使用 `_usb.CanAcceptFrame()` 限流；USB 写队列 >8KB 时按设备丢帧，只保留最新屏幕。
3. `StreamCanvasRenderer` 复用 `StyleCanvas`：数据模式渲染已保存样式、系统字体文字和实时遥测值；显示器/窗口/媒体模式渲染所选来源。两种模式互斥，不做覆盖合成。屏幕尺寸一律以设备卡片读取到的真实尺寸为准（强制覆盖样式 JSON 旧值，避免「无信号→软重启→USB 探不通」死循环）。
4. 遥测序号、屏幕采集 revision 和秒钟均未变化时跳过；数据关键帧必须收到设备
   `HICLOCK_KEYFRAME_ACK:<frameId>`，未确认时每500ms重发，确认后才进入增量更新；
   稳定期每5秒补发一次无损关键帧（或样式/来源切换强制关键帧）。
5. 显示器/窗口/媒体模式使用最高 24KB JPEG 并自适应质量（记忆上帧成功档位避免逐档降码），约 10–15 FPS；数据首帧/关键帧使用无损 RGB565 RLE，后续发送 16×16 变化块。
6. 打包 type 2 / type 5 / type 6 / type 7，1KB 分片或单 Tile + 32B 头 + 分片 CRC。
7. 使用 `_usb.TryWrite()` 写入已绑定的 COM；`InvalidateDevice` 在推进代次时使旧编码帧失效，规避 STOP ACK 与 COM 句柄重建边界的竞态。

设备设置【串流】页只展示尺寸完全匹配的内置/自定义样式，使用同一个 `StyleCanvas`
显示当前样式预览；左右按钮和下拉框修改同一 `StreamStyleId`。内置默认样式不可导出，
也不可删除；自定义样式可导出为 `.hcs-stream.json`，并可从设备设置删除。运行库不自动
灌入编辑工作区，需要编辑时显式打开 JSON 源文件。串口日志位于独立的【串口监视器】页。

#### 8.4.3 USB 写侧限流（UsbDeviceBindingService）

- `MaxQueuedScreenBytes = 4KB`，大帧按1KB写入并等待2KB低水位，避免Windows队列淹没C3 RX。
- 串口日志同时使用 `DataReceived` 和200ms主动排空；独占软重启后重发两次幂等 `HICLOCK_LOG_ON`，避免事件边沿或setup切换窗口造成日志中断。
- 大帧（≥1024B）且 `BytesToWrite > 4KB` → 拒绝写入；**不能 DiscardOutBuffer**（会从半片协议分片中截断）。
- 日志读侧与写侧**不共用 port 锁**（`AttachLogReader` 的 DataReceived 单独读），避免固件输出日志 + Studio 写整帧时双向互等导致"信号灯超时"。
- `TryWriteStop` 发送前先等当前帧完全离开 Windows 队列（最多 2s），保证分片与 `HICLOCK_STOP` 有清晰边界。

### 8.5 固件接收端实现细节

#### 8.5.1 通道选择与 RX 缓冲

- `SCREEN_STREAM_SERIAL`：USB CDC On Boot 关闭时 = `USBSerial`（Windows 枚举 COM9 等）；开启时 = `Serial` 别名。
- 任意配置在 `begin()` 前 `setRxBufferSize(SCREEN_STREAM_USB_RX_BUFFER)`，当前为 2048B（HWCDC 队列省出的内存用于把 JPEG 输入从 8KB 提高到 24KB）。Core 3.3.11 的 HWCDC 使用逐字节 FreeRTOS Queue，实际存储约为名义容量的 4 倍，因此不能直接开到 10KB；分配经 `heapEnsureForAlloc` 仲裁。

#### 8.5.2 setup 期提前探测

- `screenStreamStartSetupPollTask()`：独立任务（栈 2048，优先级 2）在 setup 期间轮询，让 Studio 无需等完整开机即可绑定设备 ID。
- 独占模式就绪后 `screenStreamStopSetupPollTask()` 停掉该任务。

#### 8.5.3 收包与重同步（screenStreamPoll → screenStreamFeedUsbByte）

- 独占模式每轮最多消费 `8` 个整包（`SCREEN_STREAM_USB_PACKETS_PER_POLL`），批量 `read()` 到 2048B 缓冲。
- 逐字节状态机：未满 4 字节时匹配魔数 `HCSF`（失败回退，**同时逐字节喂 ASCII 控制命令解析器**）；
  头 4 字节确认后不再做文本解析，避免整帧 payload 误触发命令。
- 收满 32B 头：type=4 直接进入裸流（`screenStreamBeginDirectRawFrame`）；否则按 `payloadLength` 收满整包后 `screenStreamDrawPacket`。
- 超长/畸形包直接丢弃并复位状态机。

#### 8.5.4 包分发（screenStreamDrawPacket，packet[5] 按 type）

type=2（JPEG）→ `screenStreamReceiveJpegPacket`
type=3（旧整帧）→ `screenStreamReceiveFullFramePacket`
type=5（Tile）→ `screenStreamReceiveRawTilePacket`
type=6（RGB565）→ `screenStreamReceiveRawChunkPacket`
裸流 payload → `screenStreamFeedDirectRaw`

#### 8.5.5 各帧型校验与组装要点

**通用校验**（所有类型）：
- 尺寸必须匹配屏幕（native）或为 2× 缩放的半尺寸（等比）——除 type 3 必须完全等于整帧尺寸。
- `fragmentCount` 必须等于按帧长/1KB 上取整的期望值；`payloadLength` 必须等于期望长度（末片允许不足）。
- 每片 CRC32 必须匹配；包头长度必须等于 `32 + payloadLength`。
- session/frame 序号：新 session 或更新 frame 才重置接收状态；旧帧直接拒绝。
- 收齐判定用分片位图 `mask[7]`（7×32=224 位，上限 224 片，正好覆盖 320×320×2=204800B / 1024 = 200 片）。

**type=2 JPEG 特有**：
- 头 6 个 u16 校验（尺寸/总长/分片号/负载长/分片数），最后一包必须带 `FlagLastBlock`。
- 收满后由唯一常驻的 `esp_new_jpeg` 实例按块解码，回调 `screenStreamJpegBlockToFrame` 写入帧缓冲：
  - native 尺寸 → 整行 `memcpy`；
  - 半尺寸 → 逐像素 `screenStreamWriteTransportPixel`（按比例放大 2× 到整帧）。
- 解码头、块尺寸或总输出长度校验失败 → 丢弃该帧。

**type=6 RGB565 特有**：payload 直接 memcpy 进分段缓冲（跨 32 行段边界拆分写入）；
半尺寸源 → 逐像素放大写入。

**type=5 Tile 特有**：增量帧不重传未变化块；若新帧首片到达时上一帧没收齐
（`DeltaReceived != DeltaExpected`），置 `NeedsKeyframe` 拒绝后续 tile，逼 Studio 下一帧发全帧。

#### 8.5.6 整帧缓冲（screenStreamBeginExclusiveFrameBuffer）

- 按 **32 行一段** 分 `SCREEN_STREAM_FULL_MAX_SEGMENTS=10` 段（320 行封顶），避免单次大 malloc；
  段数 = ceil(height/32)。320×320 时每段 20KB，共 200KB。
- 额外分配 24KB JPEG 输入缓冲（`SCREEN_STREAM_JPEG_MAX_BYTES`）；首次解码时按需创建共享
  `esp_new_jpeg` 句柄和 16 字节对齐块缓冲，后续帧复用，退出独占模式时释放。
- 全部 malloc 经 `heapRequestService(HEAP_SVC_SCREEN_FRAME)` + `heapEnsureForAlloc` 仲裁，失败则整帧分配回滚。

#### 8.5.7 推屏（screenStreamPresentFrame）

```
SharedSpiGuard(Tft, 30ms)   // 与 SD 卡共享 FSPI 总线互斥
waitForVSync()              // TE 引脚消隐期等待，防扫描期写 GRAM 撕裂
tft.setSwapBytes(true)
按段 pushImage(0, y, W, rows, seg)  →  dmaWait()
setSwapBytes(false)
```

- **整帧收齐后才推屏**，推屏期间不再收包（下一轮 poll 继续），从根上避免"边收边刷"的半帧闪烁。
- 1 秒窗口统计 fps / rx / decode / push 耗时；日志与帧共用 CDC，`availableForWrite() < 96` 时跳过统计，
  防止 printf 填满 TX 队列反向阻塞 RX 屏幕帧。
- 断流判定 `screenStreamIsActive()`：`LastFrameAt` 距今 >1.5s → 视为无信号（正常模式 MODE_STREAM 下显示"【无信号】"一次）。

### 8.6 独占模式生命周期

#### 8.6.1 进入

1. Studio 调 `POST /api/v1/stream/mode {enabled:true, renderer:"studio"}`。
2. 固件不读取样式，直接向 NVS（`streamboot` 分区）写入 `magic = 0x5343524E`（`SCREEN_EXCLUSIVE_RESTART_MAGIC`）及当前日志开关 → `ESP.restart()`。
3. setup 开头读 NVS：`magic` 匹配 **且** `esp_reset_reason() == ESP_RST_SW` 才置 `g_screenExclusiveMode = true`；同时恢复本次独占日志开关，随后立即删除两个一次性票据。
   - 票据一次性：掉电/重新插电/看门狗复位（非软重启）都不会误进独占。
4. 独占初始化：停 setup 轮询任务 → 显示 "HiClock" logo → `screenStreamBeginExclusiveFrameBuffer` 分配帧缓冲。
5. `setup()` 提前返回（跳过 WiFi/网络/存储/样式初始化）；`loop()` 进入专属分支：
   `screenStreamPoll()` + `taskYIELD()`（**不做 1ms 固定休眠**，避免折损 USB 小包吞吐）。

#### 8.6.2 运行

- boot logo 期间（500ms）收帧但不推屏，攒出第一帧（`RawFramePending`）后一次推屏。
- 若 500ms 内无帧 → 显示"【无信号】"。
- 无信号只保持接收，不自行软重启；未经过 Studio STOP 握手的复位会在 Windows 留下 usbser 僵尸句柄。
- 全速循环：收包 → 组装 → 推屏；FPS 由 USB 吞吐与 TFT 刷屏速度共同决定。

#### 8.6.3 退出

- Studio 发 `HICLOCK_STOP` → 固件置 `g_screenExclusiveStopRequested` → loop 延迟 50ms 后 `ESP.restart()`。
  退出由 `screenStreamExitExclusiveGracefully()` 原地退出（不再硬重启），USB `HICLOCK_STOP` 收尾。
- 重启后 NVS 票据已删 → 正常模式。
- 拔线/断电 → 自然退出（下次上电正常模式）。

### 8.7 关键设计决策

| 问题 | 方案 | 对应代码 |
|---|---|---|
| USB 丢字节导致整幅错位 | 1KB 固定分片 + 魔数重同步 + 每片 CRC + session/frame 序号 | `feedUsbByte` / `screenStreamDrawPacket` |
| 追帧积压 | 4KB准入 + 1KB分块 + 2KB低水位；显示器约10FPS | `CanAcceptFrame` / `TryWrite` |
| 半帧闪烁 | 整帧收齐才推屏；推屏与收包分轮 | `screenStreamPresentFrame` |
| 扫描期撕裂 | `waitForVSync` 等 TE 消隐 + 每段 pushImage 前对齐 | `HiClock.ino:2635` |
| 显存不足 | 32 行分段分配 + 堆仲裁器（HEAP_SVC_SCREEN_FRAME/SCREEN_STREAM） | `screenStreamBeginExclusiveFrameBuffer` |
| SD/TFT 总线互斥 | 推屏持 `SharedSpiGuard(Tft)` | `screenStreamPresentFrame` |
| 编码带宽自适应 | 显示器 JPEG 质量自适应(18-90)、上限24KB；数据关键帧无损RLE | `EncodeJpeg` / `BuildRleWireFrame` |
| 增量失效自愈 | Tile 缺块置 `NeedsKeyframe` 逼全帧 | `screenStreamReceiveRawTilePacket` |
| 增量帧丢失后停在无信号 | 设备立即发送关键帧请求，Studio 清空增量基线并重发完整帧；另有5秒周期关键帧兜底，未确认每500ms重发 | `HICLOCK_KEYFRAME_REQUIRED` / `KeyframeRequired` |
| 第二次进入串流后 COM 超时 | 重启端口静默保留：2.2 秒内不打开，随后仅允许原设备重连，防止其它历史设备抢占 | `PortRestartReservation` / `CanProbeReservedPort` |
| USB 断开后画面冻结 | 1.5 秒无新帧显示“无信号”；持续10秒先通知 Studio 释放句柄，再重启并重新枚举 USB | `HICLOCK_STREAM_TIMEOUT` / `DeviceRestarting` |
| 第一次进入正常、第二次必超时 | 每次启停先清空上一会话像素基线；预发送数据关键帧也登记为待 ACK，确认前不发送增量帧 | `InvalidateDevice` / `SendPreparedFrame` |
| 样式或显示器/窗口选择重启后丢失 | 选择变化后250ms防抖、串行原子保存设备注册表；重建来源列表前先快照已保存绑定，避免 ComboBox 双向绑定在集合 Clear 时把选择写空后回落到主屏；未枚举到已保存来源时保留原绑定 | `QueueDeviceSettingsSave` / `PrepareDeviceSettings` / `DeviceRegistry.SaveAsync` |
| 240×240 显示器/窗口模式无预览/不推流 | 显示器/窗口模式直接按设备尺寸创建运行画布，不再依赖同尺寸数据样式；缩放方式支持自动、宽、高 | `CreateSourceRuntimeCanvas` / `StreamScreenScaleMode` |
| 默认串流样式字体 | 默认样式文件保持 `embedded:` 点阵字库；编辑器打开时转成对应系统字体编辑，保存时按样式实际使用的（字体族+字号）重新栅格化生成点阵字库并写回 `default-stream-fonts.json`（未安装字体的电脑也能渲染）；自定义串流样式全程系统字体 | `EmbeddedFontRegenerator` / `ConvertEmbeddedBitmapFontsToSystemFonts` / `RestoreEmbeddedBitmapFonts` / `TrySaveStylesAsync` |
| 掉电误进独占 | NVS 一次性票据 + ESP_RST_SW 双重校验 | `HiClock.ino:4671-4679` |

### 8.8 当前保留/未启用路径

- **type=5 增量 Tile**：数据模式关键帧之后的逐秒变化使用该路径。
- **type=3 / type=4**：仅为旧版 Studio 二进制兼容，新版本不再生成。
- **Direct Raw（type=4）**：接收端仍实现（`screenStreamFeedDirectRaw`），含半尺寸逐像素放大与低字节拼接。

### 8.9 排查速查

- 串流黑屏：先看是否进入独占（串口日志 `[Screen] exclusive boot ...`）→ 再查 `[Screen] frame accepted/rejected` 尺寸日志。
- 帧被拒：日志 `frame rejected out=.. src=.. bytes=.. expected=..` 说明 Studio 画布尺寸与屏幕不匹配或字节数不符。
- 花屏/错位：CRC 通过但错位 = 尺寸字段不一致；CRC 失败 = USB 丢字节（检查线材/枚举）。
- 卡顿/追帧：观察 Studio 侧 `BytesToWrite` 是否长期 >2KB；固件侧 `fps=` 日志与 `push=` 耗时。
- 断流显示"【无信号】"：`screenStreamIsActive()` 1.5s 无新帧；检查 Studio 串流开关、已保存样式和设备设置中的显示器选择。

---

## 9. 可选监控组件与工具

把官方发行包中的下列文件放在 `HiClock Studio/tools/` 目录，构建时会自动复制到输出目录的 `tools`：

```text
LibreHardwareMonitorLib.dll
PresentMon.exe
yt-dlp.exe
```

PresentMon 的官方发行文件也可能名为 `PresentMon-版本-x64.exe`，程序能够识别该名称。

`yt-dlp.exe`（可选）用于解析 YouTube 等流媒体站点的视频直链。把官方 Windows 发行版
`yt-dlp.exe` 放入此目录即可；程序也会识别放到 PATH 上的 `yt-dlp.exe` / `yt-dlp`
（例如 `pip install yt-dlp` 或 `winget install yt-dlp.yt-dlp`）。未安装时，YouTube
解析回退到内置的纯 C# 解析器（尽力而为，部分视频可能仍无法解析）。

获取独立的 `yt-dlp.exe`（PyInstaller 单文件发行，约 3MB，无需 Python）：

- 官方 GitHub Releases：<https://github.com/yt-dlp/yt-dlp/releases/latest>
  下载 `yt-dlp.exe`（Assets 里的 Windows 独立可执行文件）。
- 或本机已有 Python 时直接生成：`python -m pip install -U yt-dlp` 后，
  把 `%LOCALAPPDATA%\Programs\Python\Python3xx\Scripts\yt-dlp.exe` 复制到本目录。

放入后构建 `HiClock Studio` 时会随 `tools\**\*` 通配符自动复制到输出目录，
运行库无需任何安装步骤即可解析 YouTube。

不要提交来源不明的 DLL/EXE。下载地址和权限说明见本文档第 13 章。

---

## 10. 手动构建

请使用已安装的 **.NET 8 SDK** 手动执行：

```powershell
cd "C:\Users\l1734\Desktop\HiClock\HiClock Studio"
dotnet restore .\HiClockStudio.sln
dotnet build .\HiClockStudio.sln -c Debug
dotnet run --project .\src\HiClockStudio.App\HiClockStudio.App.csproj
```

以上命令只构建 Windows 程序，不会编译或上传 HiClock 固件。

正式版编译运行 `HiClock Studio\编译正式版本.bat`：配置 `Release`、目标 `win-x64`、依赖
`.NET 8 Desktop Runtime`，输出到 `HiClock Studio\publish\win-x64`；脚本会先清理旧构建、
`dotnet restore`、`publish` Ring0 服务到 `tools/Ring0Service`，再 `publish` 主程序。
分发时必须连同整个 `win-x64` 文件夹一起复制，目标机需安装 .NET 8 Desktop Runtime。

---

## 11. 第一次使用

1. 打开 HiClock 单文件样式包 `.hcb` 或可编译单文件格式 `.hcs.h`。
2. 从样式列表选择样式，在 TFT 画布上拖动元素。
3. 文本元素选择“Windows 系统字体”后，可使用本机已安装的字体和像素字号。
4. 需要给固件使用时，点击左侧“字”按钮导出设备字体包：选 `.hcb` 可上传到设备免编译应用；选 `.hcs.h` 可编译单文件（含样式+字库）复制到固件工程后，由用户手动编译固件。
5. 在“设备管理”输入设备 IP 后点击“连接设备”，或扫描局域网（支持 mDNS `hiclock-XXXX.local`）。
6. 勾选一台或多台在线设备，可批量打开后台或上传当前样式。
7. 先上传并启用 `stream` 类型样式，再对目标设备开启“推送串流数据”。

当前固件没有 RAM 预览接口，所以程序不会在拖动画布时自动上传，避免反复写 LittleFS。

---

## 12. AI 样式 API

先启动 HiClock Studio，再让 AI 工具访问以下接口：

```text
GET  http://127.0.0.1:17340/api/v1/guide
POST http://127.0.0.1:17340/api/v1/ui/style-editor
GET  http://127.0.0.1:17340/api/v1/resources
GET  http://127.0.0.1:17340/api/v1/fonts
GET  http://127.0.0.1:17340/api/v1/state
GET  http://127.0.0.1:17340/api/v1/styles/current
GET  http://127.0.0.1:17340/api/v1/styles/current/preview.png
PUT  http://127.0.0.1:17340/api/v1/styles/current
POST http://127.0.0.1:17340/api/v1/styles
PUT  http://127.0.0.1:17340/api/v1/styles/{id}
POST http://127.0.0.1:17340/api/v1/actions
```

写接口的请求体是单个完整的 HiClock 样式 JSON 对象，且必须发送
`Content-Type: application/json`。推荐 AI 先读取当前样式，在保留未知字段的基础上修改，再用
`PUT /api/v1/styles/current` 写回。接口只修改 Studio 当前会话，最终仍由用户在 Studio 中保存或导出。
`/api/v1/resources` 提供数据源、元素和动作协议，`/api/v1/fonts` 返回本机真实字体，PNG 预览接口供
AI 在每轮写回后进行视觉复核；对齐、分布、编组、图层、撤销、字体和绑定等编辑器操作统一通过
`POST /api/v1/actions` 调用。

---

## 13. 第三方监控组件

程序启动时会检查可执行目录中的：

```text
LibreHardwareMonitorLib.dll
PresentMon.exe
yt-dlp.exe
```

也可以统一放入可执行目录下的 `tools` 文件夹。程序会按需动态加载，不要求 WPF 项目直接引用第三方
程序集。PresentMon 使用官方控制台参数 `--process_id`、`--output_stdout` 和独立 session name；
LibreHardwareMonitor 通过公开的 `Computer/Hardware/Sensors` 对象模型读取传感器。
当前 LibreHardwareMonitor 版本支持 PawnIO。PawnIO 驱动由用户单独安装一次，HiClock Studio
不会静默安装驱动或自行提权。检测到驱动后，CPU、主板、风扇和电压等低层传感器由 PawnIO
提供；显卡等指标仍可通过厂商 API 获取。

建议只从官方发布页获取组件：

- <https://github.com/LibreHardwareMonitor/LibreHardwareMonitor>
- <https://github.com/GameTechDev/PresentMon>
- <https://github.com/yt-dlp/yt-dlp>

首次运行前应锁定具体版本并核对对应发行包内的许可证。

`yt-dlp` 用于解析 YouTube 等站点的视频直链（可选）。程序优先查找可执行目录
`tools/yt-dlp.exe`，其次 PATH 上的 `yt-dlp.exe` / `yt-dlp`；未安装时回退到内置
纯 C# 解析器（尽力而为）。
