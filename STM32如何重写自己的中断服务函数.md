STM32如何重写自己的中断服务函数：
以`USART接发实验`中的`DEBUG_USART_IRQHandler`函数为例。
```
void DEBUG_USART_IRQHandler(void)
{
  uint8_t ucTemp;
	if(USART_GetITStatus(DEBUG_USART,USART_IT_RXNE)!=RESET)
	{		
		ucTemp = USART_ReceiveData( DEBUG_USART );
    USART_SendData(DEBUG_USART,ucTemp);    
	}	 
}	
```
由于在其他地方使能了USART接收中断，故只要USART接收到了数据就会触发中断，调用`DEBUG_USART_IRQHandler`,在该函数体中，使用if语句判断是否真的产生数据接收这个中断事件，如果是，则调用`USART_ReceiveData( DEBUG_USART );`函数将数据读取到指定存储区，再调用`    USART_SendData(DEBUG_USART,ucTemp);    
`函数将该数据发回源设备（本实验中是上位机）

参考文献：《[野火]STM32库开发实战指南》第21章