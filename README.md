# 编译提示 PA4 PE10 Unknown pin function  
	#屏蔽后不报错 这两个引脚是板载的没有引出 当使用IMU小板时才有用 

0204
移动文件后报错
Missing configuration file '/home/mat/Desktop/20-01-27/build/bebop/ap_config.h', reconfigure the project!
解决
./waf distclean 
./waf configure --board sparysuav
删除之前的编译文件重新配置编译


时钟部分报错 提示与其他部分冲突
In file included from ../../libraries/AP_HAL_ChibiOS/hwdef/common/mcuconf.h:45:0,
                 from ../../modules/ChibiOS/os/hal/ports/STM32/LLD/TIMv1/hal_st_lld.h:30,
                 from ../../modules/ChibiOS/os/hal/include/hal_st.h:46,
                 from ../../modules/ChibiOS/os/common/ports/ARMCMx/chcore_timer.h:32,
                 from ../../modules/ChibiOS/os/common/ports/ARMCMx/chcore.h:205,
                 from ../../modules/ChibiOS/os/rt/include/ch.h:110,
                 from ../../modules/ChibiOS/os/hal/osal/rt/osal.h:32,
                 from ../../modules/ChibiOS/os/hal/include/hal.h:28,
                 from ../../modules/ChibiOS/os/hal/ports/STM32/LLD/USARTv1/hal_uart_lld.c:25:
../../libraries/AP_HAL_ChibiOS/hwdef/common/stm32f47_mcuconf.h:128:0: warning: "STM32_PLLQ_VALUE" redefined
 #define STM32_PLLQ_VALUE                    8
 
In file included from ../../libraries/AP_HAL_ChibiOS/hwdef/common/chconf.h:31:0,
                 from ../../modules/ChibiOS/os/rt/include/ch.h:90,
                 from ../../modules/ChibiOS/os/hal/osal/rt/osal.h:32,
                 from ../../modules/ChibiOS/os/hal/include/hal.h:28,
                 from ../../modules/ChibiOS/os/hal/ports/STM32/LLD/USARTv1/hal_uart_lld.c:25:
./hwdef.h:38:0: note: this is the location of the previous definition
 #define STM32_PLLQ_VALUE 7
暂时先屏蔽 
# STM32_LSEDRV  以下配置获得HCLK=168M
#define STM32_LSECLK 32768U
# LSE时钟分频
# define STM32_LSEDRV (3U << 3U)	
# define STM32_PLLSRC STM32_PLLSRC_HSE
# define STM32_PLLM_VALUE 8
# define STM32_PLLN_VALUE 168
# define STM32_PLLP_VALUE 2
# define STM32_PLLQ_VALUE 7
现在比较疑惑 4.1的结构在 与3.6.8不同
	老版本modules文件夹下PX4NuttX/nuttx部分包含着MCU的BootLoad代码
		PX4Firmware
	4.1的模式变了


# ChibiOS system timer 系统定时器选用只能是TIM5/2 32bit and 自动重载
STM32_ST_USE_TIMER 5
#PA2  TIM5_CH3 TIM5 PWM(5) GPIO(25)
#PA3  TIM5_CH4 TIM5 PWM(6) GPIO(26)
#PA0  PWM_IN_5  INPUT GPIO(23)
#PA1  PWM_IN_6  INPUT GPIO(24)

ChibiOS: Done!

报错0
In file included from ../../libraries/AP_Vehicle/AP_Vehicle.h:30:0,
                 from ../../libraries/AC_AttitudeControl/AC_AttitudeControl.h:9,
                 from ../../libraries/AC_AttitudeControl/ControlMonitor.cpp:1:
../../libraries/AP_Logger/AP_Logger.h:57:2: error: #error Need HAL_BOARD_LOG_DIRECTORY for filesystem backend support
 #error Need HAL_BOARD_LOG_DIRECTORY for filesystem backend support
需要关闭 HAL_BOARD_LOG_DIRECTORY 使用

在别处clone后编译
报错1
    Failed to created ap_romfs_embedded.h 继续理解编译过程
    tool下面缺少IO_Firmware文件 上传时怎麼会漏掉呢
报错2
Command ['/usr/bin/git', 'rev-parse', '--short=8', 'HEAD'] returned 128

悲剧 在上传下载后总是各种不同的报错，继续在线下把
    仍就在 报错0开始

# Nnow some defines for logging and terrain data files.
define HAL_BOARD_LOG_DIRECTORY "/APM/LOGS"
define HAL_BOARD_TERRAIN_DIRECTORY "/APM/TERRAIN"
是能后OK

报错
../../libraries/AP_HAL_ChibiOS/system.cpp:39:1: error: static assertion failed: unexpected STM32_HCLK value
 static_assert(HAL_EXPECTED_SYSCLOCK == STM32_HCLK, "unexpected STM32_HCLK value");
 ^~~~~~~~~~~~~
compilation terminated due to -Wfatal-errors.
// static_assert(HAL_EXPECTED_SYSCLOCK == STM32_HCLK, "unexpected STM32_HCLK value");

#if defined(HAL_EXPECTED_SYSCLOCK)
#ifdef STM32_SYS_CK
static_assert(HAL_EXPECTED_SYSCLOCK == STM32_SYS_CK, "unexpected STM32_SYS_CK value");
#elif defined(STM32_HCLK)
// static_assert(HAL_EXPECTED_SYSCLOCK == STM32_HCLK, "unexpected STM32_HCLK value");
#else
#error "unknown system clock"
#endif
#endif

HAL_EXPECTED_SYSCLOCK 这个是有定义的  但是eclipse检测不到定义
STM32_SYS_CK 有定义的
STM32_HCLK 没有定义 因此是可以屏蔽的

屏蔽后编译OK
[764/766] Linking build/sparysuav/bin/arducopter
[765/766] Generating bin/arducopter.bin
[766/766] apj_gen build/sparysuav/bin/arducopter.bin
Waf: Leaving directory `/home/mat/Desktop/20-01-27/build/sparysuav'

BUILD SUMMARY
Build directory: /home/mat/Desktop/20-01-27/build/sparysuav
Target          Text     Data  BSS     Total  
----------------------------------------------
bin/arducopter  1365148  2400  194420  1561968

Build commands will be stored in build/sparysuav/compile_commands.json
'copter' finished successfully (1m27.512s)
mat@mat:~/Desktop/20-01-27$ 

保存为 20-02-10

02-10
现在编译完成，需要解决喷洒版的bootload程序问题

在更新文件到工蜂后再次编译报错
Command ['/usr/bin/git', 'rev-parse', '--short=8', 'HEAD'] returned 128


# 编译提示 PA4 PE10 Unknown pin function  
	#屏蔽后不报错 这两个引脚是板载的没有引出 当使用IMU小板时才有用 

0204
移动文件后报错
Missing configuration file '/home/mat/Desktop/20-01-27/build/bebop/ap_config.h', reconfigure the project!
解决
./waf distclean 
./waf configure --board sparysuav
删除之前的编译文件重新配置编译


时钟部分报错 提示与其他部分冲突
In file included from ../../libraries/AP_HAL_ChibiOS/hwdef/common/mcuconf.h:45:0,
                 from ../../modules/ChibiOS/os/hal/ports/STM32/LLD/TIMv1/hal_st_lld.h:30,
                 from ../../modules/ChibiOS/os/hal/include/hal_st.h:46,
                 from ../../modules/ChibiOS/os/common/ports/ARMCMx/chcore_timer.h:32,
                 from ../../modules/ChibiOS/os/common/ports/ARMCMx/chcore.h:205,
                 from ../../modules/ChibiOS/os/rt/include/ch.h:110,
                 from ../../modules/ChibiOS/os/hal/osal/rt/osal.h:32,
                 from ../../modules/ChibiOS/os/hal/include/hal.h:28,
                 from ../../modules/ChibiOS/os/hal/ports/STM32/LLD/USARTv1/hal_uart_lld.c:25:
../../libraries/AP_HAL_ChibiOS/hwdef/common/stm32f47_mcuconf.h:128:0: warning: "STM32_PLLQ_VALUE" redefined
 #define STM32_PLLQ_VALUE                    8
 
In file included from ../../libraries/AP_HAL_ChibiOS/hwdef/common/chconf.h:31:0,
                 from ../../modules/ChibiOS/os/rt/include/ch.h:90,
                 from ../../modules/ChibiOS/os/hal/osal/rt/osal.h:32,
                 from ../../modules/ChibiOS/os/hal/include/hal.h:28,
                 from ../../modules/ChibiOS/os/hal/ports/STM32/LLD/USARTv1/hal_uart_lld.c:25:
./hwdef.h:38:0: note: this is the location of the previous definition
 #define STM32_PLLQ_VALUE 7
暂时先屏蔽 
# STM32_LSEDRV  以下配置获得HCLK=168M
#define STM32_LSECLK 32768U
# LSE时钟分频
# define STM32_LSEDRV (3U << 3U)	
# define STM32_PLLSRC STM32_PLLSRC_HSE
# define STM32_PLLM_VALUE 8
# define STM32_PLLN_VALUE 168
# define STM32_PLLP_VALUE 2
# define STM32_PLLQ_VALUE 7
现在比较疑惑 4.1的结构在 与3.6.8不同
	老版本modules文件夹下PX4NuttX/nuttx部分包含着MCU的BootLoad代码
		PX4Firmware
	4.1的模式变了


# ChibiOS system timer 系统定时器选用只能是TIM5/2 32bit and 自动重载
STM32_ST_USE_TIMER 5
#PA2  TIM5_CH3 TIM5 PWM(5) GPIO(25)
#PA3  TIM5_CH4 TIM5 PWM(6) GPIO(26)
#PA0  PWM_IN_5  INPUT GPIO(23)
#PA1  PWM_IN_6  INPUT GPIO(24)

ChibiOS: Done!

报错0
In file included from ../../libraries/AP_Vehicle/AP_Vehicle.h:30:0,
                 from ../../libraries/AC_AttitudeControl/AC_AttitudeControl.h:9,
                 from ../../libraries/AC_AttitudeControl/ControlMonitor.cpp:1:
../../libraries/AP_Logger/AP_Logger.h:57:2: error: #error Need HAL_BOARD_LOG_DIRECTORY for filesystem backend support
 #error Need HAL_BOARD_LOG_DIRECTORY for filesystem backend support
需要关闭 HAL_BOARD_LOG_DIRECTORY 使用

在别处clone后编译
报错1
    Failed to created ap_romfs_embedded.h 继续理解编译过程
    tool下面缺少IO_Firmware文件 上传时怎麼会漏掉呢
报错2
Command ['/usr/bin/git', 'rev-parse', '--short=8', 'HEAD'] returned 128

悲剧 在上传下载后总是各种不同的报错，继续在线下把
    仍就在 报错0开始

# Nnow some defines for logging and terrain data files.
define HAL_BOARD_LOG_DIRECTORY "/APM/LOGS"
define HAL_BOARD_TERRAIN_DIRECTORY "/APM/TERRAIN"
是能后OK

报错
../../libraries/AP_HAL_ChibiOS/system.cpp:39:1: error: static assertion failed: unexpected STM32_HCLK value
 static_assert(HAL_EXPECTED_SYSCLOCK == STM32_HCLK, "unexpected STM32_HCLK value");
 ^~~~~~~~~~~~~
compilation terminated due to -Wfatal-errors.
// static_assert(HAL_EXPECTED_SYSCLOCK == STM32_HCLK, "unexpected STM32_HCLK value");

#if defined(HAL_EXPECTED_SYSCLOCK)
#ifdef STM32_SYS_CK
static_assert(HAL_EXPECTED_SYSCLOCK == STM32_SYS_CK, "unexpected STM32_SYS_CK value");
#elif defined(STM32_HCLK)
// static_assert(HAL_EXPECTED_SYSCLOCK == STM32_HCLK, "unexpected STM32_HCLK value");
#else
#error "unknown system clock"
#endif
#endif

HAL_EXPECTED_SYSCLOCK 这个是有定义的  但是eclipse检测不到定义
STM32_SYS_CK 有定义的
STM32_HCLK 没有定义 因此是可以屏蔽的

屏蔽后编译OK
[764/766] Linking build/sparysuav/bin/arducopter
[765/766] Generating bin/arducopter.bin
[766/766] apj_gen build/sparysuav/bin/arducopter.bin
Waf: Leaving directory `/home/mat/Desktop/20-01-27/build/sparysuav'

BUILD SUMMARY
Build directory: /home/mat/Desktop/20-01-27/build/sparysuav
Target          Text     Data  BSS     Total  
----------------------------------------------
bin/arducopter  1365148  2400  194420  1561968

Build commands will be stored in build/sparysuav/compile_commands.json
'copter' finished successfully (1m27.512s)
mat@mat:~/Desktop/20-01-27$ 

保存为 20-02-10

02-10
现在编译完成，需要解决喷洒版的bootload程序问题


在更新文件到工蜂后再次编译报错
Command ['/usr/bin/git', 'rev-parse', '--short=8', 'HEAD'] returned 128


02-16
由spaymat的hwdat生成的APbootload.bin能够被识别USB 但是在编译下载时 提示 超时，校验不通过。


