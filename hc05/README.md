# HC-05 蓝牙串口回显实验（STM32F103C8T6）

STM32CubeIDE + HAL 工程。手机通过经典蓝牙 SPP 连接 HC-05 发送字符，单片机把字符原样回发，同时在电脑串口助手上同步打印；另外每秒主动推送一条 `tick N` 心跳，用来确认链路一直通着。

## 硬件连接

| HC-05 | STM32F103C8T6 | 说明 |
| --- | --- | --- |
| TXD | PA3 (USART2_RX) | 收发交叉 |
| RXD | PA2 (USART2_TX) | 收发交叉 |
| GND | GND | 必须共地 |
| VCC | 按模块底板标注接 5V / 3.3V | 注意电平匹配 |

电脑端用 USB-TTL 接 USART1：PA9 (TX) → USB-TTL RXD，PA10 (RX) → USB-TTL TXD，同样要共地。

## 串口参数

| 串口 | 波特率 | 用途 |
| --- | --- | --- |
| USART1 | 115200 | 调试打印，`printf` 通过 `_write()` 重定向到串口 |
| USART2 | 9600 | 与 HC-05 通信，中断接收 |

## 代码要点

- `HAL_UART_RxCpltCallback()`：收到一字节后置 `ble_flag`，**并重新调用 `HAL_UART_Receive_IT()`**，否则只能收到第一个字节。
- 主循环检测 `ble_flag`：把该字节回发手机，并用 `printf("BLE recv: ...")` 打印到电脑串口。
- `HAL_GetTick()` 计时，每 1000 ms 通过 USART2 发送 `tick N`。
- `ble_flag` 用 `volatile` 修饰，因为它在中断里被修改、在主循环里读取。

## 如何打开

STM32CubeIDE → `File > Import > General > Existing Projects into Workspace`，选择本 `hc05` 目录，编译下载即可。手机端需要用支持**经典蓝牙 SPP**（不是 BLE）的蓝牙串口 App 连接 HC-05。

> 仓库只保存源码与工程配置（`.ioc`、`.cproject`、链接脚本、`Drivers/`、`Core/`）。`Debug/` 编译产物、`.settings/` 和 `*.launch` 含本机绝对路径，已由仓库根目录的 `.gitignore` 排除，clone 后重新编译即可。
