# Keil 配置 GCC 编译器用于 STM32 和 C++23 特性

我这里使用STM32F103C8T6演示
如果你想复现本文过程，你需要准备以下清单：
* Keil MDK （必需）
* 芯片支持包 Keil.STM32F1xx_DFP.2.4.0.pack （必需，请忽略版本，不要太旧即可）
* STM32CubeMX（非必需）
* STM32F103C8T6核心板（非必需）
* 任意烧录器，如DAPLink（非必需）

##  <span id="Toolchain">安装工具链 Arm GNU Toolchain arm-none-eabi </span>
[下载地址：](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)
> https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads
* 注意：下载页面会提供非常多的选项给你，这里一定不要选错，你需要注意的点：
* 1.选择Windows 不要选错其它操作系统的类型，除非你的Keil可以在Mac或者Linux上运行
* 2.选择(mingw-w64-x86_64) 而不是 (mingw-w64-i686)，这里是你电脑的CPU架构类型，选64位可以更好利用CPU资源，提高编译速度
* 3.选择AArch32，因为STM32是32位Cortex M架构
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d7a7bd14981c49f1943373033df20ade.png)

我这里选择可执行文件exe安装，当然你也可以选择下载压缩包zip直接解压到你想要的目录



## 创建Keil工程
### 前置步骤：获取依赖代码
我这里提供两种方式
#### 方式1 使用STM32CubeMX辅助生成
配置好时钟、烧录方式以及你所需的功能之后
选择 **STM32CubeIDE** 或者 **CMake** 生成代码
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8481dc366bae438db6102b92668b23bd.png)
你会问，Keil工程为什么不直接选MDK-ARM？
因为我们需要汇编启动文件**startup_stm32f103c8tx.s**以及链接脚本**STM32F103C8TX_FLASH.ld**
#### 方式2 Github
[Github获取HAL库代码](https://github.com/STMicroelectronics/stm32f1xx-hal-driver)
> https://github.com/STMicroelectronics/stm32f1xx-hal-driver
> 
### 创建Keil工程
* 步骤一 在Keil软件中创建新工程
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5bffe7f22acf4adb9bcf23f02b8ba713.png)
* 步骤二 命名后保存到前置步骤中，你放HAL库代码的地方
* 步骤三 选择芯片
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fe5376e94915450b8ee1c4d0cd33516e.png)
步骤四 这个窗口先忽略，点OK关掉它
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2367eb7fd9dd4625804906e100b0889f.png)
### 管理工程
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3f199b813bd94081801fa2064fc119a1.png)
#### 添加分组Core/Src
* 这里按照文件夹中的代码来添加分组即可
* 这里双击1把分组改名为Core/Src
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/31d9f8a6d8de476f966e129a6f1c28a0.png)
选择对应的文件，按Ctrl+A全选，然后Add添加就可以了
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c7b72b8b6b4b49d683693f98b0b905c8.png)
#### 添加分组Core/Startup
启动汇编
这里记得选择文件类型
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/560c0bb31861425ba415b07c151ebad5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d2bdc7a2e65c42799483fab24147116d.png)
#### 添加分组Drivers/STM32F1xx_HAL_Driver
这里和分组Core/Src的添加步骤一致，进入对应的文件目录后按Ctrl+A全选添加即可
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/430491628f3940b08a0f0963d2992606.png)
#### 设置GCC工具链（关键）
* 这里第三步选择我们之前安装的[GCC工具链目录（arm-none-eabi）](#Toolchain)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/283c34288d124170af5cf2d190e26740.png)

### 配置编译目标（重要）
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ae65169aef9948b4a458c5443297b8b4.png)
##### 配置编译生成的文件目录
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5c731bd8b4f74750a30fac52536fdebf.png)

#### 配置C、C++选项

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3dc66e0dce6c481b9915e7840ce247b3.png)

从上到下分别添加
* 预定义宏 Defines
> USE_HAL_DRIVER,STM32F103xB
* 头文件包含路径 Include Path（参考文件夹目录自行完成添加）
参考设置：
> .\Core\Inc
> .\Drivers\STM32F1xx_HAL_Driver\Inc
> .\Drivers\STM32F1xx_HAL_Driver\Inc\Legacy
> .\Drivers\CMSIS\Device\ST\STM32F1xx\Include;
> .\Drivers\CMSIS\Include;

如果使用了FreeTROS：

> .\Middlewares\Third_Party\FreeRTOS\Source\include
> .\Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2
> .\Middlewares\Third_Party\FreeRTOS\Source\portable\GCC\ARM_CM3

* 编译器选项 Misc Contros
>  -mcpu=cortex-m3 -mthumb -mthumb-interwork -ffunction-sections -fdata-sections -fno-common -fmessage-length=0 -specs=nosys.specs -Wl,-nostartfiles
>  
#### 配置汇编选项
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5ce68b898b30423fb8afcd1a9c37e709.png)

* 传递给汇编编译器的选项 Misc Contros
> -mcpu=cortex-m3 -mthumb -mthumb-interwork


#### 配置链接选项

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e64ce669e5b14efebcbda4e6ba08bc31.png)
* 选择链接脚本
> .\STM32F103C8TX_FLASH.ld

**.\\** 表示当前目录，即 与Keil工程文件处于同一目录下的 STM32F103C8TX_FLASH.ld 文件

* 传递给链接器的选项
> -Wl,-gc-sections,--print-memory-usage
#### 添加函数
在任意C文件中添加以下函数：
> void _init(void) {}

如果是C++文件，需要增加 **extern "C"**  修饰：
> extern "C" void _init(void) {}

## 完成创建
恭喜你，到此Keil项目已创建完毕，此时你编译会得到下图结果：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/16dac0553e734c31abd22708345ea309.png)
### 测试工程
#### 添加LED灯闪烁代码
在main函数中添加以下代码测试板载LED灯能否工作
```C
  GPIO_InitTypeDef GPIO_InitStruct = {0};
  __HAL_RCC_GPIOC_CLK_ENABLE();
  GPIO_InitStruct.Pin = GPIO_PIN_13;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
  HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);
  while(1) (
    HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
    HAL_Delay(200);
  )
```
#### 设置调试器
我这里选择DAPLink，根据你手上的调试工具选择即可
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7e13b14489d34aeaaeaa1d2ab87d85af.png)

编译并烧录后，可以看到板载LED灯在闪烁。

## 完整的工程下载
我这里提供了完整的工程。由于工具链体积较大，请移步到[ARM官网下载](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)
[点这里跳转 Github 链接]()
