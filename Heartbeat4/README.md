# Heartbeat4 — STM32F103C8T6 心跳灯

最小可用的 STM32 工程：让板载 LED 周期性翻转，用来验证「工程能编过、时钟配对了、程序真的跑起来了」这三件事。

## 功能

- PC13 板载 LED 每 100 ms 翻转一次（周期 200 ms，约 5 Hz），看起来像心跳
- 系统时钟：HSE 8 MHz 经 PLL ×9 → SYSCLK 72 MHz，AHB 72 MHz，APB1 36 MHz，APB2 72 MHz
- Flash 等待周期 2
- 只用了 GPIO，不涉及串口、定时器、中断

## 关键代码

`Core/Src/main.c` 的主循环：

```c
while (1)
{
  HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
  HAL_Delay(100);
}
```

`MX_GPIO_Init()` 把 PC13 配成推挽输出、无上下拉、低速，初始电平 `GPIO_PIN_SET`（高电平）。Blue Pill 上的 LED 是低电平点亮，所以复位后灯先是灭的。

## 在 STM32CubeIDE 里打开

1. `File → Import → General → Existing Projects into Workspace`
2. `Select root directory` 选到本目录（`Heartbeat4/`）
3. Finish，然后 `Project → Build`（锤子图标）
4. 烧录：`Run` 直接跑；`Debug`（F8）会先停在 `main` 等调试器

## 目录说明

| 路径 | 说明 |
|---|---|
| `Core/Src/main.c` | 主程序，心跳循环 |
| `Core/Inc/main.h` | 引脚与时钟相关的宏 |
| `Core/Startup/` | 启动文件（汇编，`startup_stm32f103c8tx.s`） |
| `Drivers/` | HAL 库与 CMSIS |
| `Heartbeat4.ioc` | CubeMX 配置，双击可重新生成代码 |
| `STM32F103C8TX_FLASH.ld` | 链接脚本（64 KB Flash / 20 KB RAM） |

## 两个容易踩的坑

- `Debug/`、`.settings/`、`*.launch` 不入库：前两个是本地编译产物和 IDE 配置，`.launch` 里写着本机绝对路径（源自另一台机器上的 `D:\First_week\`），换机器就失效
- 时钟依赖外部 8 MHz 晶振。如果板子没焊晶振或晶振不起振，程序会卡在 `Error_Handler()` 的死循环里，现象是**灯完全不闪**——这时把 RCC 的时钟源临时改成 HSI 再试，能闪就说明是晶振问题
