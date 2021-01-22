# STM32F407固件库编程点亮LED小灯的主要步骤
## 第一步 <font color=green>外设寄存器结构体定义</font>
条件编译：如果没有定义某个宏，就定义这个宏，并且可以防止重复编译      
**<font color=red>用结构体进行GPIO寄存器映射</font>**：外设寄存器结构体定义     
GPIO_TypeDef      
```
typedef struct
{
	__IO	uint32_t MODER;    /*GPIO模式寄存器				    地址偏移: 0x00    */
	__IO	uint32_t OTYPER;   /*GPIO输出类型寄存器				地址偏移: 0x04    */
	__IO	uint32_t OSPEEDR;  /*GPIO输出速度寄存器				地址偏移: 0x08    */
	__IO	uint32_t PUPDR;    /*GPIO上拉/下拉寄存器			地址偏移: 0x0C    */
	__IO	uint32_t IDR;      /*GPIO输入数据寄存器				地址偏移: 0x10    */
	__IO	uint32_t ODR;      /*GPIO输出数据寄存器				地址偏移: 0x14    */
	__IO	uint16_t BSRRL;    /*GPIO置位/复位寄存器 低16位部分	地址偏移: 0x18 	   */
	__IO	uint16_t BSRRH;    /*GPIO置位/复位寄存器 高16位部分	地址偏移: 0x1A     */
	__IO	uint32_t LCKR;     /*GPIO配置锁定寄存器				地址偏移: 0x1C     */
	__IO	uint32_t AFR[2];   /*GPIO复用功能配置寄存器		    地址偏移: 0x20-0x24 */
} GPIO_TypeDef;

```
**<font color=red>定义 GPIOA 寄存器结构体指针</font>**
```
/*定义 GPIOA 寄存器结构体指针*/
#define GPIOA               ((GPIO_TypeDef *) GPIOA_BASE)
```

## 第二步 <font color=green>编写端口的置位复位函数</font>
新建stm32f4xx_gpio.c文件和stm32f4xx_gpio.h文件 （外设驱动文件）
条件编译
<font color=red>在stm32f4xx_gpio.c文件中定义置位复位函数,并在stm32f4xx_gpio.h文件中声明</font>
```
/**
  *函数功能：设置引脚为高电平
  *参数说明：GPIOx，该参数为GPIO_TypeDef类型的指针，指向GPIO端口的地址
  * 			  GPIO_Pin:选择要设置的GPIO端口引脚，可输入宏GPIO_Pin_0-15，
	*										表示GPIOx端口的0-15号引脚。
  */
void GPIO_SetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
{
	/*设置GPIOx端口BSRRL寄存器的第GPIO_Pin位，使其输出高电平*/
	/*因为BSRR寄存器写0不影响，
	  GPIO_Pin只是对应位为1，其它位均为0，所以可以直接赋值*/
	
  GPIOx->BSRRL = GPIO_Pin;
}
```
在调用GPIO_SetBits时，调用格式为
`GPIO_SetBits(GPIOF,GPIO_Pin_6);`  
而`#define GPIO_Pin_6                 ((uint16_t)0x0040)  /*!< 选择Pin6 */`的意思其实就是 `1<<6`
# 另外还有一些小细节，可参考开发指南
参考文献
《1-STM32F4xx中文参考手册》  
[野火EmbedFire]《STM32库开发实战指南——基于野火霸天虎开发板》—20210107
## 第三步 <font color=green>定义外设初始化结构体和编写外设初始化函数</font>
1. <font color=red>定义外设初始化结构体</font>
```
typedef struct 
{
  uint32_t GPIO_Pin;              /*!< 选择要配置的GPIO引脚
                                        可输入 GPIO_Pin_ 定义的宏 */

  GPIOMode_TypeDef GPIO_Mode;     /*!< 选择GPIO引脚的工作模式
                                       可输入 GPIOMode_TypeDef 定义的枚举值*/

  GPIOSpeed_TypeDef GPIO_Speed;   /*!< 选择GPIO引脚的速率
                                       可输入 GPIOSpeed_TypeDef 定义的枚举值 */

  GPIOOType_TypeDef GPIO_OType;   /*!< 选择GPIO引脚输出类型
                                       可输入 GPIOOType_TypeDef 定义的枚举值*/

  GPIOPuPd_TypeDef GPIO_PuPd;     /*!<选择GPIO引脚的上/下拉模式
                                       可输入 GPIOPuPd_TypeDef 定义的枚举值*/
}GPIO_InitTypeDef;

```
<font color=red>用枚举类型定义MODER寄存器结构体,其他寄存器（PUPDR,OTYPER....）类似</font>
```
typedef enum
{ 
  GPIO_Mode_IN   = 0x00, /*!< 输入模式 */
  GPIO_Mode_OUT  = 0x01, /*!< 输出模式 */
  GPIO_Mode_AF   = 0x02, /*!< 复用模式 */
  GPIO_Mode_AN   = 0x03  /*!< 模拟模式 */
}GPIOMode_TypeDef;
```

2.<font color=red>定义GPIO_Init(函数)</font>

注意理解这一块为什么要这么写（<font color=red>稍微难理解一点</font>）：
```
for (pinpos = 0x00; pinpos < 16; pinpos++)
  {
		/*以下运算是为了通过 GPIO_InitStruct->GPIO_Pin 算出引脚号0-15*/
		
		/*经过运算后pos的pinpos位为1，其余为0，与GPIO_Pin_x宏对应。pinpos变量每次循环加1，*/
		pos = ((uint32_t)0x01) << pinpos;
   
		/* pos与GPIO_InitStruct->GPIO_Pin做 & 运算，若运算结果currentpin == pos，
		则表示GPIO_InitStruct->GPIO_Pin的pinpos位也为1，
		从而可知pinpos就是GPIO_InitStruct->GPIO_Pin对应的引脚号：0-15*/
    currentpin = (GPIO_InitStruct->GPIO_Pin) & pos;
```
其实这句`if (currentpin==pos){...}`也可以写成`if (currentpin){...}
因为
```
currentpin=(GPIO_InitStruct->GPIO_Pin) & pos;
```
的结果只可能为0000000000000000(16位)或者00...1...00(引脚号对应位为1)   
<font color=red>注：有人认为上面这段求引脚号的代码多此一举，认为`			GPIOx->MODER  &= ~(3 << (2 *pinpos));
`可以直接写成`			GPIOx->MODER  &= ~(3 << (2 *GPIO_Pin));
`这是错误的，因为虽然`	InitStruct.GPIO_Pin = GPIO_Pin_6=((uint16_t)0x0040)=(0000000001000000)2;
`,但它不等于引脚号，点亮红灯时引脚号为6</font>