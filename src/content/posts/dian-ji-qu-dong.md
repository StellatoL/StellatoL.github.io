---
title: "电机驱动"
published: 2025-01-01
description: "STM32F103C8T6是一款基于ARM Cortex-M3内核的32位微控制器，由意法半导体（STMicroelectronics）生产。其核心特性包括高达72 MHz的工作频率、64KB的Flash程序存储器和20KB的SRAM 1。这些特性使其成为各种嵌入式控制应用的理想"
tags:
  - "STM32"
  - "人人卓越"
category: "杂物"
categoryPath:
  - "杂物"
draft: false
---

# 基于STM32F103C8T6的机器人小车电机驱动与PID控制设计

## 第1节：STM32F103C8T6与机器人小车电机控制概述

### 1.1. STM32F103C8T6微控制器概览

STM32F103C8T6是一款基于ARM Cortex-M3内核的32位微控制器，由意法半导体（STMicroelectronics）生产。其核心特性包括高达72 MHz的工作频率、64KB的Flash程序存储器和20KB的SRAM 1。这些特性使其成为各种嵌入式控制应用的理想选择，尤其是在机器人技术领域。

对于电机控制而言，STM32F103C8T6的以下外设至关重要：

- **通用输入/输出（GPIO）引脚：** 该微控制器提供多达37个I/O引脚 1，可用于配置电机驱动器的方向控制信号，并与其他传感器或模块进行通信。
- **定时器（TIM）：** STM32F103C8T6内置多个通用定时器（如TIM1、TIM2、TIM3、TIM4）和高级控制定时器（部分型号）。这些定时器能够产生脉冲宽度调制（PWM）信号，这是控制直流电机速度的关键技术 1。此外，定时器还可以配置为编码器接口模式，用于读取电机转速和位置信息，这对于实现闭环控制（如PID）至关重要。

尽管STM32F103C8T6被归类为“中等密度”设备 1，但其提供的外设组合对于控制一个双电机机器人小车来说已经绰绰有余，即使是包含PID算法和编码器反馈的复杂控制系统。其72MHz的最高主频确保了控制回路能够及时执行，满足实时性要求。

<!--more-->

### 1.2. 差速驱动机器人小车

差速驱动是一种常见的机器人移动平台设计，它通过独立驱动位于共同轴线上的两个轮子来实现运动和转向 5。其基本运动方式如下：

- **前进/后退：** 两个轮子以相同的速度和相同的方向旋转。
- **转向（原地旋转/曲线转向）：** 两个轮子以不同的速度旋转，或者以相反的方向旋转。例如，要使小车左转，可以使右轮转速高于左轮，或者使左轮反转而右轮正转 6。

差速驱动的优势在于其机械结构简单，控制相对直接。

### 1.3. 电机驱动器的作用

微控制器（MCU）的GPIO引脚通常无法直接驱动直流电机，因为电机所需的电流和电压超出了MCU引脚的承受能力。因此，电机驱动器扮演了至关重要的角色。

H桥电路是电机驱动器的核心技术，它允许通过切换电路中晶体管的导通状态来改变流过电机电流的方向，从而实现电机的双向控制 7。电机驱动器将来自MCU的低功率控制信号（如方向信号和PWM信号）转换为驱动电机所需的高功率信号。这种分离不仅保护了MCU免受电机大电流的冲击，还确保了电机能够获得稳定高效的能量供应。在后续章节中，将详细介绍具体的电机驱动器IC。

## 第2节：硬件搭建与注意事项

### 2.1. 选择电机驱动器IC

为本项目选择合适的电机驱动器IC至关重要。市面上有多种选择，其中L298N和DRV8833是两种常见的适用于此项目的双H桥电机驱动器。

#### 2.1.1. L298N双H桥电机驱动器

- **描述：** L298N是一款坚固耐用、广泛应用的双H桥电机驱动器，能够独立驱动两个直流电机。它支持较宽的电压范围（最高可达35V-46V）和较大的电流（每通道最高2A）9。
- **控制逻辑：** 通常，每个电机需要2个输入引脚（如IN1, IN2）来控制方向，以及1个使能引脚（如ENA）通过PWM信号来控制速度 9。参考资料14中的表5清晰地展示了其真值表。
- **注意事项：** L298N内部晶体管导通时存在约2V的电压降 12，这会导致效率较低，并可能需要比电机额定电压更高的供电电压。在大电流工作时，通常需要加装散热片 12。许多L298N模块集成了板载5V稳压器 8。

#### 2.1.2. DRV8833双H桥电机驱动器

- **描述：** DRV8833是一款更现代、效率更高的电机驱动器，主要得益于其内部集成的低导通电阻（Rds(on)）MOSFET 15。其工作电压范围较低（2.7V - 10.8V），适用于驱动功率较小的电机（每通道RMS电流可达1.5A）15。
- **控制逻辑：** 与L298N类似，每个H桥也使用2个输入引脚（如xIN1, xIN2）进行方向和速度控制（PWM信号可施加于这些输入引脚）15。参考资料15中的表2详细说明了其控制模式。
- **特性：** DRV8833通常包含故障检测（nFAULT）和低功耗睡眠模式（nSLEEP）等额外功能 15。

#### 2.1.3. 比较与推荐

为了帮助用户根据具体需求做出选择，下表对L298N和DRV8833的关键特性进行了比较：

**表1：L298N与DRV8833电机驱动器比较**

|   |   |   |
|---|---|---|
|**特性**|**L298N**|**DRV8833**|
|工作电压范围|5V - 35V (典型模块, IC本身可达46V 10)|2.7V - 10.8V 15|
|每通道最大电流|2A (峰值3A) 11|1.5A RMS, 2A 峰值 15|
|效率 (电压降/Rds(on))|较低 (电压降约2V 12)|较高 (HS + LS Rds(on) 约 360 mΩ 15)|
|封装尺寸/集成度|通常为模块形式，体积较大|IC封装较小，模块也相对紧凑|
|散热管理|大电流时通常需要散热片 12|视具体应用而定，通常散热需求较低|
|板载5V稳压器|许多模块包含 8|通常不包含|
|特殊功能|较少|睡眠模式, 故障检测, 过流/过温保护 16|
|典型成本/易得性|成本较低，非常普及|成本略高，也较易获得|

在本报告中，**将以L298N作为详细示例**，因为它在业余爱好者套件中历史悠久且对初学者而言较为坚固。然而，DRV8833作为一种更高效的选择，尤其适用于电池供电或对体积有要求的紧凑型设计，也将被提及。

电机驱动器的选择对整个系统设计具有连锁反应。L298N的电压降可能意味着需要一个比电机名义电压更高的电池组，这会影响机器人的尺寸、重量和成本。许多L298N模块上的板载5V稳压器 8 在电机供电电压合适（例如，小于等于12V 9）时可以简化MCU的供电，但在更高电压下则可能出现问题。相比之下，DRV8833的较低工作电压范围可能更适合直接使用锂电池供电而无需额外稳压，但同时也限制了电机的选择。其高效率则有利于延长电池续航。

### 2.2. 硬件连接图：STM32F103C8T6与L298N及两路直流电机

以下是STM32F103C8T6与L298N模块及两路直流电机连接的详细说明（具体引脚选择可根据实际情况调整，此处为示例）：

- **方向控制：**
    - STM32 GPIO (例如 PA0) -> L298N IN1 (左电机方向1)
    - STM32 GPIO (例如 PA1) -> L298N IN2 (左电机方向2)
    - STM32 GPIO (例如 PA2) -> L298N IN3 (右电机方向1)
    - STM32 GPIO (例如 PA3) -> L298N IN4 (右电机方向2)
- **速度控制 (PWM)：**
    - STM32 定时器PWM输出引脚 (例如 PB0, TIM3_CH3) -> L298N ENA (左电机使能/速度)
    - STM32 定时器PWM输出引脚 (例如 PB1, TIM3_CH4) -> L298N ENB (右电机使能/速度)
- **电源：**
    - 电机电源 (例如 7.4V - 12V 电池组) -> L298N VS (或标有12V输入的端口) 和 GND
    - 电机 -> L298N OUT1/OUT2 (左电机) 和 OUT3/OUT4 (右电机)
- **逻辑电源与共地：**
    - STM32 GND -> L298N GND (共地连接至关重要 9)
    - L298N逻辑电源 (VSS或模块上的5V引脚)：
        - 如果电机电源电压小于等于12V，并且L298N模块上的5V使能跳线帽已连接，则此引脚可输出5V，可用于为STM32供电（需注意电流能力）。
        - 如果电机电源电压大于12V，应移除5V使能跳线帽，并从外部为L298N的5V逻辑输入引脚提供稳定的5V电源（例如，从STM32的5V输出引脚，如果STM32板有的话，或者使用独立的5V稳压器）8。

一个清晰的电路连接图对于正确搭建硬件至关重要，具体图示将在报告的相应部分提供（基于11中的原则）。

### 2.3. 电源注意事项

- 通常建议为MCU和电机使用独立的电源，以提高系统稳定性并防止电机噪声干扰MCU。
- 电池的选择应基于电机的额定电压和电流需求。
- STM32F103C8T6通常需要3.3V的稳定电源。“Blue Pill”等常见的STM32F103C8T6开发板通常带有板载3.3V稳压器。

### 2.4. (可选但PID控制推荐) 旋转编码器

- 旋转编码器是一种反馈设备，用于测量电机的转速和/或位置 20。
- 常见的类型是增量式编码器，它输出A、B两相正交脉冲信号。
- **接线：** 编码器的A、B相输出连接到STM32的定时器输入引脚（配置为编码器接口模式）或配置为外部中断的GPIO引脚。编码器还需要VCC和GND供电 20。

尽早引入编码器的概念，即使用户最初不打算实现PID控制，也能为后续的高级控制打下基础，并强调闭环系统反馈的重要性。

## 第3节：STM32CubeIDE电机控制配置

### 3.1. 在STM32CubeIDE中设置新工程

STM32CubeIDE是一款集成了STM32CubeMX图形化配置工具的集成开发环境。创建一个新工程的步骤通常包括选择目标MCU型号（STM32F103C8T6），然后利用STM32CubeMX进行引脚和外设的配置 7。

### 3.2. 系统时钟配置

系统时钟（SYSCLK）是MCU执行指令和外设工作的核心。对于STM32F103C8T6，通常将其配置为最大频率72MHz。如果开发板上装有外部高速晶振（HSE），例如“Blue Pill”板上常见的8MHz晶振 24，则应优先使用HSE作为时钟源，并通过PLL倍频到72MHz。同时，需要确保APB1和APB2总线上的外设时钟（PCLK1, PCLK2）也得到正确配置，因为定时器等外设挂载在这些总线上，其工作频率与总线时钟相关 7。

### 3.3. L298N方向控制的GPIO配置

为L298N的IN1、IN2、IN3、IN4引脚选择四个STM32F103C8T6的GPIO引脚（例如PA0, PA1, PA2, PA3）。在STM32CubeMX中，将这些引脚配置为“GPIO_Output”模式 7。可以设置初始输出电平（例如，低电平），输出速度。对于推挽输出模式，通常不需要配置上拉或下拉电阻。如果需要更深入地理解GPIO寄存器级别的配置，可以参考STM32参考手册中关于GPIOx_CRL和GPIOx_CRH寄存器的说明 4，但STM32 HAL库对这些底层细节进行了抽象。

### 3.4. PWM速度控制的定时器配置

选择STM32F103C8T6上的两个定时器通道用于产生PWM信号，以控制L298N的ENA和ENB引脚（例如，TIM2的通道1和通道2，或者不同定时器的通道，如TIM1的通道1和TIM2的通道1）。所选的GPIO引脚必须是相应定时器通道的PWM输出复用功能引脚 1。

在STM32CubeMX中进行如下配置 4：

1. **选择并使能定时器：** 例如，选择TIM2。
2. **设置时钟源：** 选择“Internal Clock”。
3. **配置通道为PWM输出模式：** 例如，将Channel 1和Channel 2配置为“PWM Generation CH1”和“PWM Generation CH2”。
4. **参数设置：**
    - **预分频器 (Prescaler, PSC)：** 用于降低定时器的计数时钟频率。计数器时钟频率 fCK_CNT​ 的计算公式为：fCK_CNT​=fAPB_Timer​/(PSC+1)，其中 fAPB_Timer​ 是定时器所在的APB总线时钟频率。
    - **计数模式 (Counter Mode)：** 对于PWM生成，通常选择向上计数模式。
    - **计数周期 (Counter Period, Auto-Reload Register - ARR)：** 定义PWM信号的周期，从而决定PWM频率。PWM频率 fPWM​ 的计算公式为：fPWM​=fCK_CNT​/(ARR+1)。
    - **脉冲宽度 (Pulse, Capture Compare Register - CCRx)：** 定义PWM信号的占空比。占空比 DutyCycle 的计算公式为：DutyCycle=(CCRx/(ARR+1))×100%。
    - **PWM模式 (PWM Mode)：** 通常选择“PWM Mode 1”或“PWM Mode 2”。
        - PWM Mode 1：当 TIMx_CNT<TIMx_CCRx 时，输出有效电平（例如高电平）。
        - PWM Mode 2：当 TIMx_CNT>TIMx_CCRx 时，输出有效电平（例如高电平）。

PWM频率和分辨率的考量：

PWM频率的选择并非随意。较高的频率可以使电机运行更平稳，减少可闻噪音，但也可能增加电机驱动器的开关损耗。较低的频率可能对电机本身的效率不利。对于L298N，1kHz至20kHz范围内的频率较为常见，例如9中提到了1.5kHz。这是一个需要权衡的参数。PSC和ARR的组合不仅决定PWM频率，也影响PWM的分辨率。较大的ARR值可以提供更精细的占空比控制。

示例计算：

假设APB1定时器时钟为72MHz（假设HCLK未分频直接供给APB1总线上的定时器）。

期望PWM频率为10kHz。

- 方案一：设置PSC = 0，则 fCK_CNT​=72MHz。此时，ARR=(72MHz/10kHz)−1=7200−1=7199。
- 方案二：设置PSC = 71，则 fCK_CNT​=72MHz/(71+1)=1MHz。此时，ARR=(1MHz/10kHz)−1=100−1=99。 方案一具有更高的分辨率。

完成配置后，从STM32CubeMX生成初始化代码。

### 3.5. 表2：STM32F103C8T6与L298N控制引脚映射示例

|   |   |   |   |
|---|---|---|---|
|**STM32 引脚**|**STM32 功能**|**L298N 引脚**|**用途**|
|PA0|GPIO_Output|IN1|左电机方向控制1|
|PA1|GPIO_Output|IN2|左电机方向控制2|
|PA2|GPIO_Output|IN3|右电机方向控制1|
|PA3|GPIO_Output|IN4|右电机方向控制2|
|PB0|TIM3_CH3 (PWM)|ENA|左电机速度控制|
|PB1|TIM3_CH4 (PWM)|ENB|右电机速度控制|
|3.3V|-|(可选，若L298N模块需要外部5V逻辑供电且STM32有5V输出，则连接至L298N的5V输入，否则L298N的5V由其板载稳压器提供或外部独立5V供电)|L298N逻辑电源|
|GND|-|GND|电源地|

此表提供了一个清晰的参考，帮助用户正确连接硬件，并理解MCU各引脚在软件中的角色，从而将软件配置与物理接线联系起来。

## 第4节：实现机器人小车基本运动 (C代码与HAL库)

### 4.1. 工程结构与HAL初始化

由STM32CubeIDE生成的`main.c`文件包含了程序的主体结构。在`main()`函数中，首先会调用`HAL_Init()`进行HAL库的初始化，接着调用`SystemClock_Config()`配置系统时钟。之后，由STM32CubeMX生成的GPIO和定时器初始化函数（如`MX_GPIO_Init()`和`MX_TIMx_Init()`）也会被调用。

### 4.2. 启动PWM信号

为了使能L298N的速度控制引脚ENA和ENB，需要在相应的定时器初始化之后启动PWM信号。这通过为每个PWM通道调用`HAL_TIM_PWM_Start()`函数来实现 7。此操作通常在初始化阶段执行一次。

C

```c
// 假设htim3用于PWM输出，通道3和4分别控制左电机和右电机
HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_3); // 启动左电机PWM
HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_4); // 启动右电机PWM
```

### 4.3. 定义电机和方向类型 (示例)

为了提高代码的可读性和可维护性，建议使用`typedef enum`来定义电机和方向的类型：

C

```c
typedef enum {
    MOTOR_LEFT,
    MOTOR_RIGHT
} Motor_TypeDef;

typedef enum {
    DIR_FORWARD,
    DIR_BACKWARD,
    DIR_STOP // 电机停止（通过方向控制引脚）
} Direction_TypeDef;
```

### 4.4. 底层电机控制函数

#### 4.4.1. 设置电机PWM速度函数

此函数用于设置指定电机PWM信号的占空比，从而控制电机转速。

C

```c
/**
  * @brief  设置电机PWM速度
  * @param  htimx: 定时器句柄指针 (例如 &htim3)
  * @param  TIM_CHANNEL_y: 定时器通道 (例如 TIM_CHANNEL_3)
  * @param  speed_percentage: 速度百分比 (0-100)
  * @retval None
  */
void set_motor_pwm_speed(TIM_HandleTypeDef *htimx, uint32_t TIM_CHANNEL_y, uint8_t speed_percentage) {
    uint16_t raw_pwm;
    if (speed_percentage > 100) {
        speed_percentage = 100;
    }
    // 根据ARR值计算CCRx的原始值
    raw_pwm = (uint16_t)(((float)speed_percentage / 100.0f) * htimx->Instance->ARR);
    __HAL_TIM_SET_COMPARE(htimx, TIM_CHANNEL_y, raw_pwm);
}
```

此函数接收定时器句柄、定时器通道和期望的速度百分比作为参数。它首先将速度百分比转换为对应定时器ARR（自动重载寄存器）值的原始PWM比较值（CCRx），然后使用`__HAL_TIM_SET_COMPARE()`宏函数来更新PWM的占空比 25。

#### 4.4.2. L298N电机方向控制函数

此函数根据L298N的控制逻辑，通过设置相应的GPIO引脚电平来控制电机方向。

C

```c
// 假设已在CubeMX中为方向引脚定义了宏，例如:
// #define L_IN1_GPIO_Port GPIOA
// #define L_IN1_Pin GPIO_PIN_0
// #define L_IN2_GPIO_Port GPIOA
// #define L_IN2_Pin GPIO_PIN_1
// #define R_IN3_GPIO_Port GPIOA
// #define R_IN3_Pin GPIO_PIN_2
// #define R_IN4_GPIO_Port GPIOA
// #define R_IN4_Pin GPIO_PIN_3

/**
  * @brief  设置L298N驱动的电机方向
  * @param  motor: 选择电机 (MOTOR_LEFT 或 MOTOR_RIGHT)
  * @param  direction: 选择方向 (DIR_FORWARD, DIR_BACKWARD, 或 DIR_STOP)
  * @retval None
  */
void set_motor_direction_l298n(Motor_TypeDef motor, Direction_TypeDef direction) {
    if (motor == MOTOR_LEFT) {
        if (direction == DIR_FORWARD) {
            HAL_GPIO_WritePin(L_IN1_GPIO_Port, L_IN1_Pin, GPIO_PIN_SET);   // IN1 = High
            HAL_GPIO_WritePin(L_IN2_GPIO_Port, L_IN2_Pin, GPIO_PIN_RESET); // IN2 = Low
        } else if (direction == DIR_BACKWARD) {
            HAL_GPIO_WritePin(L_IN1_GPIO_Port, L_IN1_Pin, GPIO_PIN_RESET); // IN1 = Low
            HAL_GPIO_WritePin(L_IN2_GPIO_Port, L_IN2_Pin, GPIO_PIN_SET);   // IN2 = High
        } else { // DIR_STOP (快速制动)
            HAL_GPIO_WritePin(L_IN1_GPIO_Port, L_IN1_Pin, GPIO_PIN_RESET); // IN1 = Low
            HAL_GPIO_WritePin(L_IN2_GPIO_Port, L_IN2_Pin, GPIO_PIN_RESET); // IN2 = Low
        }
    } else if (motor == MOTOR_RIGHT) {
        if (direction == DIR_FORWARD) {
            HAL_GPIO_WritePin(R_IN3_GPIO_Port, R_IN3_Pin, GPIO_PIN_SET);   // IN3 = High
            HAL_GPIO_WritePin(R_IN4_GPIO_Port, R_IN4_Pin, GPIO_PIN_RESET); // IN4 = Low
        } else if (direction == DIR_BACKWARD) {
            HAL_GPIO_WritePin(R_IN3_GPIO_Port, R_IN3_Pin, GPIO_PIN_RESET); // IN3 = Low
            HAL_GPIO_WritePin(R_IN4_GPIO_Port, R_IN4_Pin, GPIO_PIN_SET);   // IN4 = High
        } else { // DIR_STOP (快速制动)
            HAL_GPIO_WritePin(R_IN3_GPIO_Port, R_IN3_Pin, GPIO_PIN_RESET); // IN3 = Low
            HAL_GPIO_WritePin(R_IN4_GPIO_Port, R_IN4_Pin, GPIO_PIN_RESET); // IN4 = Low
        }
    }
}
```

此函数使用`HAL_GPIO_WritePin()`函数 23 来设置L298N的INx引脚电平，遵循L298N的真值表 14。注意，L298N的“停止”可以通过将两个INx引脚都设为低电平（快速制动）或将使能引脚（ENA/ENB）设为低电平（自由滑行停止）来实现。上述函数实现了快速制动。

### 4.5. 高层机器人运动函数

基于底层的速度和方向控制函数，可以封装出更高级别的机器人整体运动函数。这种分层设计提高了代码的模块化程度和可重用性。如果将来更换电机驱动器（例如，从L298N换到DRV8833），主要修改可能仅限于底层的方向控制函数，而高层API（如`car_forward()`）可以保持不变，这大大增强了代码的可维护性和适应性。

C

```c
// 假设htim_L和htim_R是分别用于左右电机的定时器句柄
// 假设TIM_CHANNEL_L和TIM_CHANNEL_R是对应的PWM通道
extern TIM_HandleTypeDef htim_L; // 例如 htim3
extern TIM_HandleTypeDef htim_R; // 例如 htim3
#define TIM_CHANNEL_L TIM_CHANNEL_3 // 示例
#define TIM_CHANNEL_R TIM_CHANNEL_4 // 示例

/**
  * @brief  小车前进
  * @param  speed_percentage: 速度百分比 (0-100)
  * @retval None
  */
void car_forward(uint8_t speed_percentage) {
    set_motor_direction_l298n(MOTOR_LEFT, DIR_FORWARD);
    set_motor_direction_l298n(MOTOR_RIGHT, DIR_FORWARD);
    set_motor_pwm_speed(&htim_L, TIM_CHANNEL_L, speed_percentage);
    set_motor_pwm_speed(&htim_R, TIM_CHANNEL_R, speed_percentage);
}

/**
  * @brief  小车后退
  * @param  speed_percentage: 速度百分比 (0-100)
  * @retval None
  */
void car_backward(uint8_t speed_percentage) {
    set_motor_direction_l298n(MOTOR_LEFT, DIR_BACKWARD);
    set_motor_direction_l298n(MOTOR_RIGHT, DIR_BACKWARD);
    set_motor_pwm_speed(&htim_L, TIM_CHANNEL_L, speed_percentage);
    set_motor_pwm_speed(&htim_R, TIM_CHANNEL_R, speed_percentage);
}

/**
  * @brief  小车左转 (原地旋转)
  * @param  speed_percentage: 速度百分比 (0-100)
  * @retval None
  */
void car_turn_left_pivot(uint8_t speed_percentage) {
    set_motor_direction_l298n(MOTOR_LEFT, DIR_BACKWARD);
    set_motor_direction_l298n(MOTOR_RIGHT, DIR_FORWARD);
    set_motor_pwm_speed(&htim_L, TIM_CHANNEL_L, speed_percentage);
    set_motor_pwm_speed(&htim_R, TIM_CHANNEL_R, speed_percentage);
}

/**
  * @brief  小车右转 (原地旋转)
  * @param  speed_percentage: 速度百分比 (0-100)
  * @retval None
  */
void car_turn_right_pivot(uint8_t speed_percentage) {
    set_motor_direction_l298n(MOTOR_LEFT, DIR_FORWARD);
    set_motor_direction_l298n(MOTOR_RIGHT, DIR_BACKWARD);
    set_motor_pwm_speed(&htim_L, TIM_CHANNEL_L, speed_percentage);
    set_motor_pwm_speed(&htim_R, TIM_CHANNEL_R, speed_percentage);
}

/**
  * @brief  小车停止 (通过PWM占空比为0实现，方向控制引脚保持上次状态或设为制动)
  * @retval None
  */
void car_stop_pwm() {
    set_motor_pwm_speed(&htim_L, TIM_CHANNEL_L, 0);
    set_motor_pwm_speed(&htim_R, TIM_CHANNEL_R, 0);
    // 可选: 执行快速制动
    // set_motor_direction_l298n(MOTOR_LEFT, DIR_STOP);
    // set_motor_direction_l298n(MOTOR_RIGHT, DIR_STOP);
}
```

这些高层函数调用了前面定义的底层函数来实现小车的各种运动状态 7。转向策略可以有多种，例如原地旋转（一个轮子前进，另一个轮子后退）或曲线转向（一个轮子比另一个轮子转得快）5。

### 4.6. 主循环测试

在`main()`函数的`while(1)`循环中，可以编写简单的测试代码来验证这些运动函数的正确性，例如：

C

```c
int main(void) {
    //... HAL_Init(), SystemClock_Config(), MX_GPIO_Init(), MX_TIM3_Init()...
    // 假设htim_L 和 htim_R 已正确初始化并赋值为 &htim3
    // 假设TIM_CHANNEL_L 和 TIM_CHANNEL_R 已正确定义

    HAL_TIM_PWM_Start(&htim_L, TIM_CHANNEL_L);
    HAL_TIM_PWM_Start(&htim_R, TIM_CHANNEL_R);

    while (1) {
        car_forward(50);    // 前进，速度50%
        HAL_Delay(2000);    // 持续2秒
        car_stop_pwm();     // 停止
        HAL_Delay(1000);    // 停止1秒

        car_turn_left_pivot(30); // 左转，速度30%
        HAL_Delay(1000);    // 持续1秒
        car_stop_pwm();     // 停止
        HAL_Delay(1000);    // 停止1秒
        
        car_backward(50);   // 后退，速度50%
        HAL_Delay(2000);
        car_stop_pwm();
        HAL_Delay(1000);

        car_turn_right_pivot(30); // 右转，速度30%
        HAL_Delay(1000);
        car_stop_pwm();
        HAL_Delay(1000);
    }
}
```

## 第5节：PID控制增强性能简介

### 5.1. 为何需要PID？开环控制的局限性

第4节中实现的基本运动控制属于“开环控制”。这意味着MCU向电机发出指令（例如，设置一定的PWM占空比），但并不关心电机是否真正达到了期望的速度，或者小车是否真的在直线行驶。

开环控制的性能会受到多种因素的影响，例如：

- 电池电压波动：电池电量下降时，相同的PWM占空比可能无法产生相同的电机转速。
- 地面摩擦力变化：在不同表面行驶时，电机负载会变化。
- 电机个体差异：即使是同型号的电机，其特性也可能略有不同。
- 负载变化：小车上装载物品的重量变化。

为了克服这些局限性，引入了“闭环控制”策略，其中PID（比例-积分-微分）控制器是最常用的一种。PID控制器通过使用传感器反馈来持续监测系统状态，并根据期望状态与实际状态之间的误差来调整控制输出，从而提高系统的性能和鲁棒性 21。

### 5.2. PID基本理论解释

PID控制器的核心思想是根据当前误差、累积误差和误差变化率来计算控制量。

- **误差 (Error, e(t)):** 期望设定点（例如，目标速度）与实际测量过程变量（例如，编码器测得的当前速度）之间的差值 21。
- **比例 (Proportional, P) 项 (Kp​⋅e(t)):**
    - 控制器输出与当前误差成正比。误差越大，P项产生的校正作用也越大。
    - **作用：** 加快系统响应速度，减小稳态误差，但通常无法完全消除稳态误差。过大的Kp​会导致系统振荡甚至不稳定 21。
- **积分 (Integral, I) 项 (Ki​⋅∫e(t)dt):**
    - 累积过去的误差。只要存在误差，I项就会持续作用，直到误差被消除。
    - **作用：** 消除系统的稳态误差。过大的Ki​会导致积分饱和（积分 विंडअप），引起系统超调和振荡，响应变慢 21。
- **微分 (Derivative, D) 项 (Kd​⋅dtde(t)​):**
    - 根据误差的变化率来预测未来的误差趋势，并提前进行校正。
    - **作用：** 减小超调，抑制振荡，改善系统稳定性，加快稳定过程。D项对过程变量中的噪声非常敏感，过大的Kd​或噪声较大的系统可能导致输出剧烈波动 21。

PID控制律公式：

控制器的输出 Output(t) 由这三项加权求和得到：

Output(t)=Kp​e(t)+Ki​∫0t​e(τ)dτ+Kd​dtde(t)​

其中，Kp​, Ki​, Kd​ 分别是比例、积分和微分增益系数，需要根据具体系统进行整定 21。

理解P、I、D各项的独立作用及其组合效应是PID参数整定的基础。每个参数都针对系统响应的不同方面进行调整。PID控制器本质上是试图基于当前和过去的状态来预测未来并进行修正，从而使系统能够自适应地达到期望状态。

### 5.3. 反馈的必要性：引入编码器

为了使PID控制器能够工作，必须能够测量“过程变量”（例如电机速度）。旋转编码器是电机控制中常用的传感器，用于提供这种反馈 20。编码器产生的脉冲信号可以被转换成电机的速度或位置信息。下一节将详细介绍如何在STM32F103C8T6上集成和读取编码器数据。

## 第6节：STM32F103C8T6集成编码器

### 6.1. 理解正交编码器

正交编码器（或称A/B相编码器）通常有两个输出通道，分别称为A相和B相。这两个通道输出方波脉冲信号，并且A相和B相信号之间存在90度的相位差。通过检测这两个信号的脉冲数量和相位关系，可以确定电机旋转的距离（或角度）和方向 20。编码器的一个关键参数是每转脉冲数（Pulses Per Revolution, PPR），它表示电机轴旋转一周时，A相或B相输出的脉冲个数 33。

### 6.2. 配置STM32定时器的编码器模式

STM32微控制器的通用定时器（例如STM32F103上的TIM1, TIM2, TIM3, TIM4，具体型号需查阅数据手册确认是否支持编码器接口）可以配置为“编码器接口模式”，从而硬件解码编码器信号 20。

**STM32CubeMX 设置步骤 20:**

1. **选择合适的定时器：** 例如，选择TIM3。
2. **使能编码器模式：** 在定时器的“Combined Channels”配置中，选择“Encoder Mode”。
3. **配置编码器接口参数：**
    - **Encoder Mode：** 选择“Encoder Mode TI1 and TI2”。这将使用定时器的两个输入通道（TI1FP1和TI2FP2）分别连接编码器的A相和B相信号。
    - **Counter Period (ARR)：** 对于16位定时器，通常将ARR设置为最大值65535（0xFFFF），以允许在计数器溢出前回转较大范围。也可以根据期望的计数行为进行配置。
    - **Input Filter (ICx Filter)：** 可以为输入通道配置数字滤波器，以减少外部噪声对编码器信号的干扰。
4. **GPIO引脚配置：** 连接到编码器A相和B相的GPIO引脚（例如，TIM3_CH1和TIM3_CH2对应的引脚）将自动配置为定时器输入复用功能。
5. **启动定时器：** 在代码中，使用 `HAL_TIM_Encoder_Start(&htim_encoder, TIM_CHANNEL_ALL);` 启动定时器的编码器模式 20。

一旦配置完成并启动，定时器的计数器寄存器（TIMx->CNT）会根据编码器的旋转自动增加或减少。

使用硬件编码器模式比通过外部中断手动处理编码器脉冲要高效得多，尤其是在电机转速较高时。它将计数和方向检测的任务卸载到硬件，从而释放CPU资源 [39 (Scott Seidman的评论), 20]。

### 6.3. 读取编码器计数并计算速度

#### 6.3.1. 读取原始计数值

启动编码器模式后，可以通过读取定时器的计数器寄存器来获取原始计数值：

current_counts = __HAL_TIM_GET_COUNTER(&htim_encoder); 20。

#### 6.3.2. 计算速度 (例如，每采样时间的脉冲数或RPM)

速度的计算需要周期性地采样编码器的计数值。可以使用另一个定时器产生周期性中断（例如，每10ms或100ms）来进行速度计算。

1. 在一个固定的采样时间间隔 Δt 内，读取编码器计数值的变化量 Δcounts：
   
    Δcounts=current_counts−previous_counts
    
    需要处理计数器溢出/下溢的情况，特别是当ARR值不是足够大或者位置信息发生回绕时 20。对于16位计数器和ARR=65535，当从0xFFFF变为0x0000（正转）或从0x0000变为0xFFFF（反转）时，需要特殊处理差值。
    
2. 将脉冲数转换为转速（例如，RPM - 每分钟转数）：
   
    SpeedRPS​=(Δcounts/PPRencoder​)/Δtseconds​ (转/秒)
    
    SpeedRPM​=SpeedRPS​×60 (转/分钟)
    
    其中，PPRencoder​ 是编码器每转产生的脉冲数，Δtseconds​ 是采样时间间隔（以秒为单位）。
    
    例如，34 提供了一个RPM计算公式：rpm=(timer_counter/encoder_counts_per_rev)×60.0。这里的 timer_counter 对应于 Δcounts，encoder_counts_per_rev 对应于 PPRencoder​，并且假设这个计算是在1秒的时间间隔内完成的。
    
    36 中描述的M方法公式为 n=M0​/(C⋅T0​)，其中 M0​ 是 Δcounts，C 是 PPRencoder​，T0​ 是采样时间 Δtseconds​。
    

#### 6.3.3. C代码结构示例 (编码器读取与速度计算)

C

```c
// 编码器数据结构
typedef struct {
    TIM_HandleTypeDef* htim;         // 编码器定时器句柄
    int16_t            previous_counter_value; // 上一次的计数值 (假设为16位定时器)
    int32_t            current_position;       // 累积的位置 (可选)
    float              current_speed_rps;    // 当前速度 (转/秒)
    uint16_t           ppr;                  // 编码器每转脉冲数
    uint32_t           last_update_tick;     // 上次更新速度的时间戳 (ms)
    float              sample_time_seconds;  // 速度计算的采样周期 (秒)
} Encoder_TypeDef;

// 示例：在定时器中断服务程序中更新速度
// 假设此函数由一个周期性定时器中断调用，例如每10ms
void Encoder_UpdateSpeed(Encoder_TypeDef* encoder) {
    uint32_t current_tick = HAL_GetTick();
    int16_t current_counter = (int16_t)__HAL_TIM_GET_COUNTER(encoder->htim);

    // 计算计数值差值，考虑16位计数器溢出
    int16_t diff = current_counter - encoder->previous_counter_value;
    // 简单的溢出处理：如果差值绝对值大于半个计数范围，则认为发生了溢出
    // (0xFFFF / 2 = 32767)
    // if (diff > 30000) { // 假设反向溢出 (e.g., 0 -> 65530)
    //     diff -= 65536;
    // } else if (diff < -30000) { // 假设正向溢出 (e.g., 65530 -> 0)
    //     diff += 65536;
    // }
    // 更稳健的溢出处理可能需要更复杂的逻辑或依赖于定时器更新事件

    float time_elapsed_seconds = (float)(current_tick - encoder->last_update_tick) / 1000.0f;

    if (time_elapsed_seconds >= encoder->sample_time_seconds) { // 确保达到采样周期
        if (encoder->ppr > 0 && time_elapsed_seconds > 0.0001f) { // 防止除以零
            encoder->current_speed_rps = ((float)diff / (float)encoder->ppr) / time_elapsed_seconds;
        } else {
            encoder->current_speed_rps = 0.0f;
        }
        encoder->previous_counter_value = current_counter;
        encoder->last_update_tick = current_tick;
        
        // 可选：累积位置
        // encoder->current_position += diff;
    }
}
```

速度计算的准确性和频率直接影响PID控制的性能。一个充满噪声的速度信号（例如，来自低PPR编码器或更新频率不足）会使PID参数整定变得困难，并可能导致系统不稳定，尤其是当微分项（Kd）增益较高时。因此，选择合适的编码器和设计恰当的速度计算周期非常重要。

## 第7节：实现电机PID速度控制 (C代码与HAL库)

### 7.1. PID数据结构定义

为每个需要PID控制的电机定义一个C结构体，用于存储PID参数和相关的状态变量 21。

C

```c
typedef struct {
    float Kp;                   // 比例增益
    float Ki;                   // 积分增益
    float Kd;                   // 微分增益

    float setpoint;             // 目标设定值 (例如，期望速度 RPS)
    float last_error;           // 上一次的误差
    float integral_sum;         // 积分累计值
    float integral_max;         // 积分累计值的上限 (用于抗积分饱和)
    float integral_min;         // 积分累计值的下限

    float output_min;           // PID输出值的下限 (例如，PWM占空比的最小值0)
    float output_max;           // PID输出值的上限 (例如，PWM占空比的最大值100)
    
    // 对于基于时间的PID计算
    // uint32_t last_compute_time;  // 上次计算PID的时间戳
    // float sample_time_seconds;   // PID计算周期 (秒)
} PID_Controller_TypeDef;

// 为左右电机分别实例化PID控制器
PID_Controller_TypeDef pid_left_motor;
PID_Controller_TypeDef pid_right_motor;
```

这种结构化方法有助于管理多个PID控制器的参数和状态，例如在双电机驱动的机器人小车中，左右轮通常需要独立的PID控制器 21。

### 7.2. PID初始化函数 `pid_init()`

创建一个函数来初始化PID控制器的参数，包括增益系数、设定点（可以后续修改）、输出限制、积分限制和采样时间。同时，重置误差和积分累积值。

C

```c
/**
  * @brief  初始化PID控制器参数
  * @param  pid: 指向PID控制器结构体的指针
  * @param  Kp: 比例增益
  * @param  Ki: 积分增益
  * @param  Kd: 微分增益
  * @param  setpoint: 初始目标设定值
  * @param  out_min: PID输出下限
  * @param  out_max: PID输出上限
  * @param  integral_limit: 积分累积值的绝对值上限
  * @retval None
  */
void pid_init(PID_Controller_TypeDef* pid, float Kp, float Ki, float Kd, 
              float setpoint, float out_min, float out_max, float integral_limit) {
    pid->Kp = Kp;
    pid->Ki = Ki;
    pid->Kd = Kd;
    pid->setpoint = setpoint;
    pid->output_min = out_min;
    pid->output_max = out_max;
    pid->integral_max = integral_limit;
    pid->integral_min = -integral_limit; // 对称的积分限制

    pid->last_error = 0.0f;
    pid->integral_sum = 0.0f;
    // pid->last_compute_time = HAL_GetTick(); // 如果使用基于时间戳的dt
}
```

此初始化函数参考了21中的`set_pid`和37中的`PID`构造函数的设计思想。

### 7.3. PID计算函数 `pid_compute()`

此函数将周期性地被调用（例如，每隔一个固定的`sample_time_seconds`），根据当前测量值计算PID输出。

C

```c
/**
  * @brief  执行PID计算
  * @param  pid: 指向PID控制器结构体的指针
  * @param  current_value: 当前测量值 (例如，当前电机速度 RPS)
  * @param  dt: 离散时间间隔 (两次计算之间的时间差，单位：秒)
  * @retval float: PID计算得到的输出值 (例如，PWM占空比百分比)
  */
float pid_compute(PID_Controller_TypeDef* pid, float current_value, float dt) {
    if (dt <= 0.0f) { // 防止dt为零或负数导致计算错误
        return pid->output_min; // 或返回上一次的有效输出
    }

    // 1. 计算误差
    float error = pid->setpoint - current_value;

    // 2. 比例项
    float P_term = pid->Kp * error;

    // 3. 积分项 (带抗积分饱和)
    pid->integral_sum += pid->Ki * error * dt;
    if (pid->integral_sum > pid->integral_max) {
        pid->integral_sum = pid->integral_max;
    } else if (pid->integral_sum < pid->integral_min) {
        pid->integral_sum = pid->integral_min;
    }
    float I_term = pid->integral_sum; // 在某些实现中，I_term = Ki * integral_sum

    // 4. 微分项
    float derivative = (error - pid->last_error) / dt;
    float D_term = pid->Kd * derivative;
    
    // 更新上一次误差，用于下次微分计算
    pid->last_error = error;

    // 5. 计算总输出
    float output = P_term + I_term + D_term;

    // 6. 输出限幅
    if (output > pid->output_max) {
        output = pid->output_max;
    } else if (output < pid->output_min) {
        output = pid->output_min;
    }

    return output;
}
```

此计算函数参考了21的`apply_pid`和37的`PID_Compute`以及21中的PID实现逻辑。在积分项和微分项的计算中，`dt`（时间间隔）应该是两次PID计算之间的_实际_经过时间，而不是简单地使用配置的采样时间。这可以通过`HAL_GetTick()`来测量，以获得更准确的控制，尤其是在控制循环的定时不是绝对精确的情况下。

### 7.4. 将PID输出应用于PWM占空比

`pid_compute()`函数返回的`output`值（例如，范围在0到100之间，如果`output_min/max`如此设置）需要被映射到控制电机PWM的占空比范围（例如，对于定时器的ARR值为999，则PWM原始值范围为0-999）。

如果PID的输出设计为可以为负值（例如，当PID同时控制方向和速度时），则输出的符号可以决定电机方向，其绝对值决定PWM大小。但在本报告中，我们假设方向由高层函数（如`car_forward`）独立控制，PID仅输出一个正值（0-100）来控制速度大小。

可以使用第4节中定义的set_motor_pwm_speed()函数来应用PID计算得到的PWM占空比：

set_motor_pwm_speed(&htim_motor, TIM_CHANNEL_MOTOR, (uint8_t)pid_output);

其中pid_output是pid_compute返回的经过限幅的控制量。

### 7.5. 带PID的主控制循环

1. 初始化左右电机的PID控制器（调用`pid_init()`）。
2. 初始化编码器，并启动周期性的速度计算（例如，通过定时器中断）。
3. 在一个周期性循环中（例如，由一个专用定时器中断触发，中断周期为PID的`sample_time_seconds`）： a. 从编码器获取左右电机的当前速度（`current_speed_L`, `current_speed_R`）。 b. 计算自上次PID计算以来经过的实际时间`dt`。 c. 根据用户输入或更高级别的导航逻辑，设置左右电机的期望速度（`pid_left_motor.setpoint`, `pid_right_motor.setpoint`）。 d. 为左电机计算PID输出：`pwm_L = pid_compute(&pid_left_motor, current_speed_L, dt);` e. 为右电机计算PID输出：`pwm_R = pid_compute(&pid_right_motor, current_speed_R, dt);` f. 根据期望的运动方向（例如，如果设定点为负，则方向为反向，但这需要PID输出和方向逻辑的配合），设置电机方向。为简化起见，这里假设PID输出始终为正（0-100），方向由`car_forward()`等函数预先设定。 g. 将计算得到的`pwm_L`（经过适当转换）应用到左电机的PWM控制引脚（ENA）。 h. 将计算得到的`pwm_R`（经过适当转换）应用到右电机的PWM控制引脚（ENB）。

PID控制回路、编码器读数和PWM更新必须仔细定时和同步。使用硬件定时器中断来执行主PID控制循环，可以确保一致的执行时序，这对于PID控制的稳定性至关重要。这种系统集成确保了控制器能够基于最新的反馈信息做出及时响应。

## 第8节：PID控制器参数整定

### 8.1. 整定的目标

PID参数整定的目标是找到一组Kp​,Ki​,Kd​值，使得控制系统对于特定的机器人小车能够表现出稳定且良好的动态性能：快速的响应时间、最小的超调量、较短的稳定时间以及零稳态误差（或在可接受范围内）32。

### 8.2. 手动整定方法 (试凑法)

这是一种常用的、基于经验的整定方法 30：

1. **从P控制器开始：** 将Ki​和Kd​设为0。
2. **调整Kp​：** 从一个较小的值开始，逐渐增加Kp​。观察系统响应（例如，电机速度达到设定值的过程）。目标是获得一个相对较快且没有持续振荡的响应。如果系统开始出现持续振荡或变得不稳定，则减小Kp​。通常将Kp​设置在系统开始轻微振荡的临界值的50%-70%左右。
3. **加入I控制器：** 在选定的Kp​值基础上，从一个较小的值开始逐渐增加Ki​。Ki​的作用是消除稳态误差。观察系统是否能够最终达到设定点。如果Ki​过大，可能会导致超调增加、响应变慢或产生振荡（积分饱和）。如果需要，可以回头微调Kp​。
4. **加入D控制器 (如果需要)：** 如果系统在P和I控制下仍存在较大的超调或振荡，可以尝试加入一个较小的Kd​值来抑制这些现象，提高系统的阻尼。Kd​对系统噪声敏感，因此不宜设置过大。在许多速度控制应用中，PD或PI控制器可能就足够了，不一定需要完整的PID。
5. **迭代优化：** PID参数整定通常是一个迭代的过程。对一个参数的调整可能会影响其他参数的最佳值。需要耐心观察和反复试验。

### 8.3. Kp​,Ki​,Kd​对系统响应的影响

下表总结了各PID参数对系统响应特性的主要影响 21：

**表3：PID参数整定指南**

|   |   |   |   |
|---|---|---|---|
|**参数**|**增大该参数的影响**|**减小该参数的影响**|**参数过大的常见问题**|
|Kp​|加快响应速度，减小稳态误差，增加系统振荡性|减慢响应速度，增大稳态误差，减小系统振荡性|剧烈超调，持续振荡，系统不稳定|
|Ki​|消除稳态误差，可能增加超调和稳定时间|稳态误差较大，响应可能更快（如果Ki​之前过大）|积分饱和，超调过大，响应缓慢，系统振荡|
|Kd​|减小超调，缩短稳定时间，提高系统稳定性，对噪声敏感|增加超调，延长稳定时间，可能降低稳定性|放大高频噪声，导致输出抖动，系统不稳定|

此表为用户在整定过程中提供了一个快速参考，帮助理解参数调整的后果并解决常见的整定问题。

### 8.4. 机器人小车PID整定实用技巧

- **独立测试：** 首先在机器人轮子离地或机器人被架空的情况下，对单个电机的速度控制环进行整定。
- **参数迁移：** 一个电机整定完成后，可以将得到的参数作为另一个电机PID控制器的初始参数。由于电机和机械结构可能存在差异，通常还需要对第二个电机进行微调。
- **实地测试：** 在机器人实际运行的地面上进行最终测试和调整，因为地面摩擦力会影响系统响应。
- **数据记录与可视化：** 通过串口将设定点、当前速度、PID输出等数据发送到PC端，并使用串口助手或绘图工具（如STM32CubeIDE的SWV功能 20，或自定义的绘图软件）进行可视化。这有助于直观地分析系统响应曲线，从而更有效地进行参数调整。
- **差速驱动的协调：** 对于差速驱动机器人，要实现直线行驶，两轮的PID控制器都需要良好整定，并且两轮的设定速度必须相等。由于机械差异，可能需要为左右轮设置略微不同的PID增益。更高级的控制策略可能包括一个上层控制器，用于校正航向偏差（例如，使用IMU反馈）或确保两轮行驶距离一致（例如，通过PID控制两轮编码器计数的差值 38）。
- **注意安全：** 在整定过程中，电机可能会突然高速旋转或反向，确保机器人处于安全的环境中，避免损坏或伤人。

### 8.5. Ziegler-Nichols整定法 (简述)

Ziegler-Nichols方法是一种启发式的PID参数整定方法，适用于手动整定较为困难的系统 30。它通常包括两个步骤：首先，在纯比例控制（Ki​=0,Kd​=0）下，找到使系统产生持续等幅振荡的临界比例增益Ku​和此时的振荡周期Tu​。然后根据Ku​和Tu​的值，通过经验公式计算出Kp​,Ki​,Kd​的初始值。

## 第9节：总结与展望

### 9.1. 成果总结

本报告详细阐述了如何使用STM32F103C8T6微控制器设计和实现一个双轮差速驱动机器人小车的电机控制系统。内容涵盖了STM32CubeIDE的配置、L298N电机驱动器的接口与控制、通过PWM实现电机调速、集成旋转编码器以获取速度反馈，以及PID控制理论的讲解和C语言实现。通过本报告的学习，用户应能够：

- 理解STM32F103C8T6在电机控制中的关键外设（GPIO、TIM）。
- 掌握电机驱动器的基本原理和使用方法。
- 实现小车的基本运动控制（前进、后退、转向）。
- 配置STM32定时器读取编码器数据并计算速度。
- 理解PID控制算法的原理，并能将其应用于电机速度闭环控制。
- 初步掌握PID参数的手动整定方法。

### 9.2. 完整示例代码结构 (概念性)

一个完整的项目代码通常会包含以下模块化的文件结构：

- `main.c`: 主程序文件，包含初始化调用和主循环。
- `motor_control.h` / `motor_control.c`: 包含电机底层控制函数（方向、PWM设置）和高层运动函数（前进、后退等）。
- `encoder.h` / `encoder.c`: 包含编码器初始化、数据读取和速度计算相关的函数。
- `pid_controller.h` / `pid_controller.c`: 包含PID数据结构定义、PID初始化和PID计算函数。
- `stm32f1xx_it.c`: 中断服务程序文件，可能包含用于编码器速度计算或PID控制循环的定时器中断处理函数。

### 9.3. 后续增强建议

本项目为更复杂的机器人应用奠定了坚实的基础。以下是一些可能的后续增强方向：

- **避障功能：** 集成超声波传感器或红外传感器，实现障碍物检测和规避。
- **循迹功能：** 使用红外反射传感器阵列，使小车能够沿着预设的黑线或白线行驶。
- **精确转向控制：** 利用编码器反馈，实现基于特定角度或半径的精确转弯 5。
- **IMU（惯性测量单元）集成：** 加入陀螺仪和加速度计，用于获取小车的姿态信息（俯仰、滚转、偏航），可用于更稳定的运动控制或平衡机器人等应用。
- **无线遥控：** 通过蓝牙模块（如HC-05/06）或Wi-Fi模块（如ESP8266/ESP32）实现对小车的远程控制。
- **SLAM（即时定位与地图构建）：** 对于更高级的自主导航，可以探索SLAM算法的实现（这通常需要更强大的处理器和更复杂的传感器）。

通过本项目所学习到的微控制器编程、传感器集成和控制理论知识，具有高度的可迁移性，能够为开发者在机器人技术及相关嵌入式系统领域的深入探索提供有力支持。
