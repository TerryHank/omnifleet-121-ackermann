# 121 号阿克曼底盘公开快照

采集日期为 2026-11-15（中国标准时间）。`stm32/` 包含当前 STM32F407 源码、GCC 构建脚本、固件及此前调试版本。`nx/` 包含车端当前的 ROS 底盘驱动、Xbox 服务、网页控件、生效配置和 SBUS 转发服务。更大的工作区和数据见 GitHub Releases。

## 运行配置

- J4／PC8 是转向 PWM，M1 是左驱动轮，M2 是右驱动轮。
- 固件转向上限为 ±60°。归档时前端和 Xbox 共用的车端软件上限为 ±55°，STM32 读出的零点偏置为 −30°。角度指令尚须对照实车转向几何标定。
- 实体 SBUS 由 STM32 转发到 NX 的 /stm32/sbus。CH3 负责前后，CH1 负责左右，CH9 的 1000～2000 µs 对应 0.05～3.00 m/s 的请求速度，步长 0.05 m/s。CH9 收到原始值 1700 时，请求速度档为 2.85 m/s。
- NX 上的 SBUS 服务向 /stm32/sbus/cmd_vel 发布 Twist。固件每轮限幅 1.5 转/秒，按轮径 100 mm 计算，直行轮周速度指令上限约 0.47 m/s。
- 原 Xbox 方案的左摇杆负责前后，右摇杆负责转向，十字键上下以 0.05 m/s 步长调档。Xbox 代码随快照归档，采集时车端启用 SBUS 服务。
- ROS 2 Humble 接收 `/cmd_vel`。J4 的新目标在下一次 PWM 周期生效。偏置保存在 STM32，转向软件上限保存在车端 JSON 配置。

## STM32

从实车 Flash 回读确认当前程序与 `stm32/images/ackermann121-j4-60deg.bin` 相符。该文件长 71184 字节，写入地址为 `0x08000000`。STM32 最后一个 Flash 扇区（`0x08060000` 起）保存偏置校准。

在 Linux 或 WSL 下安装 Arm GNU Toolchain 并进入 `stm32/`，运行 `python3 gcc/build.py` 和 `bash gcc/link.sh` 可生成 ELF、HEX 与 BIN。源文件保留上游许可与版权声明。`diagnostics/check_ackermann_early.c` 是早期转向范围验证的原始记录。

## 下载工作区

```sh
gh release download snapshot-2026-11-15 --repo TerryHank/omnifleet-121-ackermann
sha256sum -c SHA256SUMS
```

各 `.tar.zst` 文件可使用 `tar --zstd -xf 文件名` 解开。NX 工作区附带采集时的安装目录与构建产物，路径和架构与原车有关。恢复时按目标机器的目录、ROS 环境和设备配置核对服务配置。

公开快照的筛选统计见 `nx/archive-summary.json`。账号和凭据、原 Git 对象库、明确标为专有的依赖包、未确认再分发许可的模型、原始录制文件以及完整 MCU Flash 转储保留在源端。应用固件镜像、非敏感配置、数据和验证日志随快照发布。
