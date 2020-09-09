---
title: 软件定时器FREERTOS篇10
categories: STM32 RTOS FREE_RTOS
tags: STM32 RTOS FREE_RTOS
description: RTOS软件定时器
---
# 软件定时器解析
## 软件定时器结构体分析

```c
/* The definition of the timers themselves. */
typedef struct tmrTimerControl /* The old naming convention is used to prevent breaking kernel aware debuggers. */
{
	const char				*pcTimerName;		/*<< Text name.  This is not used by the kernel, it is included simply to make debugging easier. */ /*lint !e971 Unqualified char types are allowed for strings and single characters only. */
	ListItem_t				xTimerListItem;		/*<< Standard linked list item as used by all kernel features for event management. */
	TickType_t				xTimerPeriodInTicks;/*<< How quickly and often the timer expires. */
	void 					*pvTimerID;			/*<< An ID to identify the timer.  This allows the timer to be identified when the same callback is used for multiple timers. */
	TimerCallbackFunction_t	pxCallbackFunction;	/*<< The function that will be called when the timer expires. */
	#if( configUSE_TRACE_FACILITY == 1 )
		UBaseType_t			uxTimerNumber;		/*<< An ID assigned by trace tools such as FreeRTOS+Trace */
	#endif
	uint8_t 				ucStatus;			/*<< Holds bits to say if the timer was statically allocated or not, and if it is active or not. */
} xTIMER;
```
- 软件定时器名字`pcTimerName`
- 软件定时器列表项`xTimerListItem`
- 软件定时器的周期`xTimerPeriodInTicks`
- 软件定时器ID`pvTimerID`
- 软件定时器的回调函数`pxCallbackFunction`
- 软件定时器状态`ucStatus`

## 常用函数
- 创建软件定时器

```c
TimerHandle_t xTimerCreate(const char * const pcTimerName,  /*lint !e971 Unqualified char types are allowed for strings and single characters only. */
                           const TickType_t xTimerPeriodInTicks,
                           const UBaseType_t uxAutoReload,
                           void * const pvTimerID,
                           TimerCallbackFunction_t pxCallbackFunction )
```
- 控制软件定时器`BaseType_t xTimerGenericCommand(TimerHandle_t xTimer,const BaseType_t xCommandID,const TickType_t xOptionalValue,BaseType_t * const pxHigherPriorityTaskWoken,const TickType_t xTicksToWait)`
- 启动软件定时器`BaseType_t xTimerStart(TimerHandle_t xTimer,const TickType_t xTicksToWait)`
- 停止软件计时器`BaseType_t xTimerStop(TimerHandle_t xTimer, TickType_t xTicksToWait)`
- 更改计时器周期`BaseType_t xTimerChangePeriod(TimerHandle_t xTimer,TickType_t xNewPeriod,TickType_t xTicksToWait)`
- 删除软件定时器`BaseType_t xTimerDelete(TimerHandle_t xTimer, TickType_t xTicksToWait)`
- 重启软件定时器`BaseType_t xTimerReset(TimerHandle_t xTimer,TickType_t xTicksToWait)`
- 中断中启动软件定时器`BaseType_t xTimerStartFromISR(TimerHandle_t xTimer xTimer,BaseType_t *pxHigherPriorityTaskWoken)`
- 中断中停止软件计时器`BaseType_t xTimerStopFromISR(TimerHandle_t xTimer,BaseType_t *pxHigherPriorityTaskWoken)`
- 中断中更改计时器周期`BaseType_t xTimerChangePeriodFromISR(TimerHandle_t xTimer,TickType_t xNewPeriod,BaseType_t *pxHigherPriorityTaskWoken)`
- 中断中重启软件定时器`BaseType_t xTimerResetFromISR(TimerHandle_t xTimer,BaseType_t *pxHigherPriorityTaskWoken)`

# 添加库函数
- 打开 `Manage Run-Time Environment`
- 选择 `RTOS->Timers`

# 软件定时器
- 修改`FreeRTOSConfig.h`

```c
/* Software timer definitions. */
#define configUSE_TIMERS                      1
#define configTIMER_TASK_PRIORITY             configMAX_PRIORITIES-1
#define configTIMER_QUEUE_LENGTH              10
#define configTIMER_TASK_STACK_DEPTH          (configMINIMAL_STACK_SIZE * 2)
```

- 修改`main.c`

```c
#include "FreeRTOS.h"
#include "task.h"
#include "timers.h"
#include "my_gpio.h"
#include "my_usart.h"


/* 任务句柄 */
static TaskHandle_t AppTaskCreate_Handle = NULL;/* 创建任务句柄 */

/* 句柄 */
static TimerHandle_t Swtmr1_Handle =NULL;   /* 软件定时器句柄 */
static TimerHandle_t Swtmr2_Handle =NULL;   /* 软件定时器句柄 */

static uint32_t TmrCb_Count1 = 0; /* 记录软件定时器1回调函数执行次数 */
static uint32_t TmrCb_Count2 = 0; /* 记录软件定时器2回调函数执行次数 */

static void AppTaskCreate(void);/* 用于创建任务 */

static void Swtmr1_Callback(void* parameter);
static void Swtmr2_Callback(void* parameter);

static void BSP_Init(void);/* 用于初始化板载相关资源 */
int main(void)
{
    BSP_Init();
    printf("FreeRTOS-TIMERS!\r\n");
    BaseType_t xReturn = pdPASS;/* 定义一个创建信息返回值，默认为pdPASS */
    /* 创建 AppTaskCreate 任务 */
    xReturn = xTaskCreate((TaskFunction_t	)AppTaskCreate,		//任务函数
                          (const char* 	)"AppTaskCreate",		//任务名称
                          (uint32_t 		)128,	//任务堆栈大小
                          (void* 		  	)NULL,				//传递给任务函数的参数
                          (UBaseType_t 	)1, 	//任务优先级
                          (TaskHandle_t*  )&AppTaskCreate_Handle);	//任务控制块

    if (pdPASS == xReturn)/* 创建成功 */
        vTaskStartScheduler();   /* 启动任务，开启调度 */
    else
        return -1;
    while(1);   /* 正常不会执行到这里 */
}

/**
 * @brief  为了方便管理，所有的任务创建函数都放在这个函数里面
 * @param  无
 * @retval 无
 */
static void AppTaskCreate(void)
{
    taskENTER_CRITICAL();           //进入临界区
    Swtmr1_Handle=xTimerCreate((const char*		)"AutoReloadTimer",
                               (TickType_t			)1000,/* 定时器周期 1000(tick) */
                               (UBaseType_t		)pdTRUE,/* 周期模式 */
                               (void*				  )1,/* 为每个计时器分配一个索引的唯一ID */
                               (TimerCallbackFunction_t)Swtmr1_Callback);
    if(NULL != Swtmr1_Handle)
        xTimerStart(Swtmr1_Handle,0);	//开启周期定时器

    Swtmr2_Handle=xTimerCreate((const char*			)"OneShotTimer",
                               (TickType_t			)5000,/* 定时器周期 5000(tick) */
                               (UBaseType_t			)pdFALSE,/* 单次模式 */
                               (void*					  )2,/* 为每个计时器分配一个索引的唯一ID */
                               (TimerCallbackFunction_t)Swtmr2_Callback);
    if(NULL != Swtmr2_Handle)
        xTimerStart(Swtmr2_Handle,0);	//开启周期定时器

    vTaskDelete(AppTaskCreate_Handle); //删除AppTaskCreate任务

    taskEXIT_CRITICAL();            //退出临界区
}
/**
 * @brief  Swtmr1_Callback主体
 * @param  parameter 参数
 * @retval 无
 */
static void Swtmr1_Callback(void* parameter)
{		
  TickType_t tick_num1;

  TmrCb_Count1++;						/* 每回调一次加一 */

  tick_num1 = xTaskGetTickCount();	/* 获取滴答定时器的计数值 */
  
  LED1_TOGGLE;
  
  printf("Swtmr1_Callback  %d \n", TmrCb_Count1);
  printf("TICK=%d\n", tick_num1);
}
/**
 * @brief  Swtmr2_Callback主体
 * @param  parameter 参数
 * @retval 无
 */
static void Swtmr2_Callback(void* parameter)
{	
  TickType_t tick_num2;

  TmrCb_Count2++;						/* 每回调一次加一 */

  tick_num2 = xTaskGetTickCount();	/* 获取滴答定时器的计数值 */

  printf("Swtmr2_Callback %d \n", TmrCb_Count2);
  printf("TICK=%d\n", tick_num2);
}
/**
 * @brief  板级外设初始化，所有板子上的初始化均可放在这个函数里面
 * @param  无
 * @retval 无
 */
static void BSP_Init(void)
{
    /*
     * STM32中断优先级分组为4，即4bit都用来表示抢占优先级，范围为：0~15
     * 优先级分组只需要分组一次即可，以后如果有其他的任务需要用到中断，
     * 都统一用这个优先级分组，千万不要再分组，切忌。
     */
    NVIC_PriorityGroupConfig( NVIC_PriorityGroup_4 );

    /* LED 初始化 */
    LED_GPIO_Config();

    /* 串口初始化	*/
    USART_Config();
    /* 按键初始化 */
    Key_GPIO_Config();
}

```
# 调试
- 编译下载到开发板
- 每1000 个tick 时候软件定时器就会触发一次回调函数
- 当5000 个tick 到来的时候,触发软件定时器单次模式的回调函数,之后便不会再次调用了
