# π RuView（中文版）

[English](README.md) | **简体中文**

<p align="center">
  <a href="https://cognitum.one/seed">
    <img src="assets/ruview-hero-h3-v3.gif" alt="RuView：通过 WiFi 实时感知姿态、呼吸与心率的可视化演示" width="100%">
  </a>
</p>

> RuView 将普通 WiFi 信号转化为空间感知数据，用于研究和探索人员存在、运动及生命体征等信息，无需摄像头。

[用户指南](docs/user-guide.md) · [构建指南](docs/build-guide.md) · [ESP32 固件](firmware/esp32-csi-node/README.md) · [架构决策](docs/adr/README.md)

## 项目简介

WiFi 无线电波会受到环境和人体活动的影响。RuView 捕获并分析 WiFi 信道状态信息（CSI）等射频数据，提供一套边缘侧感知与空间智能工具。项目包含 Rust 感知服务、ESP32 传感节点固件、模型与训练工具、可视化界面，以及面向开发者和 AI 工具的辅助组件。

可探索的方向包括：

- **存在与活动感知**：分析人员存在、占用和运动等信号特征。
- **生命体征研究**：从信号中提取呼吸和心率相关估计值。
- **姿态与空间建模**：研究基于 CSI 的姿态估计、环境指纹和多视角融合。
- **边缘部署**：将 CSI 采集和部分信号处理运行在 ESP32 等设备上。
- **集成与自动化**：通过 API、WebSocket、MQTT 等方式连接可视化和智能家居系统。

## 重要说明：能力边界与安全

RuView 仍在积极开发中。不同数据源、硬件、房间环境和模型的能力与成熟度并不相同；请查阅对应文档中的限制和验证状态，不要仅凭演示或模拟数据推断实际效果。

- 呼吸、心率、姿态、人数和存在检测结果**不是医疗测量或安全认证结果**，也不应单独用于急救、诊断或安全控制。
- 构建成功或模拟器运行成功不等于真实硬件已验证。硬件验证需要目标设备上的运行证据。
- RSSI（接收信号强度）模式能力有限；完整 CSI 感知需要兼容硬件，例如 ESP32 节点或研究型网卡。
- 准确率和性能数字应以项目提供的证据、测试条件及复现方法为准；实验性、合成或尚未验证的能力不能当作已验证功能。
- 请妥善保护 WiFi 密码、API token、设备密钥、原始 CSI 和个人数据；不要将这些信息提交到仓库。

## 快速开始

### Docker 演示（无需传感硬件）

以下方式使用模拟数据启动服务和界面。容器需要 API token：

```bash
docker pull ruvnet/wifi-densepose:latest
export RUVIEW_API_TOKEN="$(openssl rand -hex 32)"
docker run --rm \
  -p 127.0.0.1:3000:3000 \
  -e RUVIEW_API_TOKEN \
  ruvnet/wifi-densepose:latest
```

然后访问 <http://localhost:3000>。模拟数据用于体验软件流程，不代表真实环境中的感知效果。

### 运行确定性参考验证

在仓库根目录执行：

```bash
python archive/v1/data/proof/verify.py
```

该脚本用于验证参考信号处理流程的确定性。通过此验证不代表任何特定硬件或感知能力已通过现场验证。

### 从源码构建 Rust 工作区

请先阅读[构建指南](docs/build-guide.md)和[用户指南](docs/user-guide.md)，并安装项目所需的 Rust 工具链及系统依赖。基本命令如下：

```bash
cd v2
cargo build --release
cargo test --workspace --no-default-features
```

实际可构建的 crate、所需系统库和可用功能以当前源码及相关文档为准。

## 实时硬件

项目提供 ESP32 CSI 节点固件。安装固件前，请先阅读[固件说明](firmware/esp32-csi-node/README.md)及设备对应的发布说明，确认芯片型号、闪存容量、固件包和烧录步骤完全匹配。不要将不同芯片或不同固件包的文件混用。

首次体验建议先使用 Docker 模拟模式；连接真实 ESP32 节点时，再按照[用户指南中的 ESP32 配置](docs/user-guide.md#esp32-s3-full-csi)设置节点、感知服务及 UDP 来源限制。网络接入请遵循文档中的绑定地址和来源白名单说明，不要在不可信网络上开放未经保护的数据端口。

## 工作原理（概览）

```text
WiFi / 射频信号
       ↓
兼容传感设备采集 CSI 等信号数据
       ↓
信号预处理与特征提取
       ↓
模型推理、融合或实验性分析
       ↓
API、WebSocket、界面或集成组件
```

具体处理流程取决于数据来源、所用硬件和模型。部分功能仍处于实验或研究阶段，请参阅具体组件文档和验证记录。

## 项目目录

| 路径 | 说明 |
| --- | --- |
| `v2/` | 当前 Rust 工作区及相关 crate |
| `archive/v1/` | Python 参考实现与确定性证明文件 |
| `firmware/esp32-csi-node/` | ESP32 CSI 节点固件与操作文档 |
| `docs/` | 用户指南、构建指南、集成文档及架构决策 |
| `harness/ruview/` | RuView 开发者工具与操作 harness |
| `harness/homecore/` | Homecore 开发者 harness |
| `plugins/ruview/` | RuView 插件与相关资源 |

## 文档

- [用户指南](docs/user-guide.md)：安装、数据来源、服务运行、API 和硬件配置。
- [构建指南](docs/build-guide.md)：验证、源码构建、Docker、可视化和固件构建。
- [ESP32 CSI 固件文档](firmware/esp32-csi-node/README.md)：硬件要求、固件安装、配置及限制。
- [Home Assistant 集成](docs/integrations/home-assistant.md)：MQTT 等集成说明。
- [RuView harness](harness/ruview/README.md)：开发者和自动化工具说明。
- [Homecore harness](harness/homecore/README.md)：Homecore 开发者工具说明。
- [架构决策记录（ADR）](docs/adr/README.md)：设计背景和决策记录。
- [版本与固件发布](https://github.com/ruvnet/RuView/releases)：查找对应版本的发行包及校验信息。

## 参与贡献

欢迎通过问题反馈和 Pull Request 参与项目。提交前请阅读仓库中的 [`AGENTS.md`](AGENTS.md) 和 [`CLAUDE.md`](CLAUDE.md)，遵循其中的安全、证据标注和验证要求。请勿提交凭据、原始个人数据或未经审查的生成文件。

## 许可证

项目主许可证为 MIT，详情见 [`LICENSE`](LICENSE)；个别组件可能有额外许可声明，请一并查看对应目录中的说明。

## 支持

- [GitHub Issues](https://github.com/ruvnet/RuView/issues)
- [GitHub Discussions](https://github.com/ruvnet/RuView/discussions)
- [PyPI：wifi-densepose](https://pypi.org/project/wifi-densepose/)

---

**RuView — 以射频信号探索空间感知。**
