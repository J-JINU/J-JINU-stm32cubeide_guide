UART는 rs232또는 rs485를 사용할 때 사용한다.
UART와 USART는 둘 다 async모드로 사용할 수 있으므로 UART/USART가 혼용되어 작성되어있더라도 양해바란다.

그리고 dma를 사용하지 않을 것이라면 인터럽트를 이용하여 non-blocking되도록 코드를 사용하여야한다. 인터럽트를 이용하는 것보다 dma를 사용하여 처리하는 것을 강력하게 추천한다.

## UART MODE
모드가 다양하게 존재하지만 Asynchronous이외의 모드는 특수한 인터페이스 구성이 아니라면 사용할 일이 없다.

Async 모드를 선택하면 아래에 Flow Control 항목이 활성화된다. Disable하면된다.

## Parameter Setting
일반적인 설정은 아래와 같다.
1. BaudRate : 9600/19200/38400/57600/115200을 주로 사용한다. 더 높게 사용할 수도 있으나 상대편의 인터페이스와 협의가 필요하다.
2. WordLength : 8bit
3. Parity : none
4. stopbit : 1

 * 위의 셋팅을 하면 초당 전송 byte수는 baudrate / 10으로 계산할 수 있다.
 * EX. baudrate :9600bps -> 실제 byte전송 속도는 960Bps가 된다.

## FIFO Mode
- enable 하면 8바이트 FIFO 버퍼가 활성화된다. 모아서 한 번에 반복문으로 처리하는 것이 더 빠를 것이라는 것을 뇌는 알고 있으나 실제로 데이터가 몇 byte가 들어올지 모르는 경우가 더 많을 것이라고 생각하므로 굳이 써야하나 싶다.

## Interrupt
개인적으로 UART사용시 DMA 구성은 TX만 DMA 처리하고 RX는 인터럽트로 처리하는 것을 선호한다.
따라서 NVIC탭에서 UART항목에 체크한다.

## RX
* RX에서 DMA를 사용하지는 않는것을 선호하는 편이다. 이유는 몇 byte가 들어올지 확정짓기가 힘들기 때문이다 당연히 모아서 한번에 반복문 돌리는 것이 성능적으로는 좋다는 것을 뇌는 알고 있다.
 
RX interrupt를 사용하려면 아래의 코드를 추가한다.
```c
LL_USART_EnableIT_RXNE(UARTx or USARTx);
```
RX interrupt는 별도의 flag clear가 없고 FIFO 버퍼에 값을 읽어서 비워야 clear 된다 
``` c
LL_USART_IsActiveFlag_RXNE(UARTx or USARTx)
```
버퍼에 데이터가 들어오면 위의 코드가 true가 된다.
이때 아래의 코드를 사용하면 버퍼의 데이터를 비우게 된다.
```c
tmp = LL_USART_ReceiveData8(UARTx);
```

만약 loopback 테스트를 하고싶다면 아래의 코드를 사용하면된다.
```c
LL_USART_TransmitData8(UART5, LL_USART_ReceiveData8(UART5));
```
RX interrupt관련 전체 코드는 아래와 같다.

```c
/*UART와 관련된 인터럽트 발생시 전부 여기서 처리해야함*/
void UARTx_IRQHandler(void)
{
  /* USER CODE BEGIN UART5_IRQn 0 */
  if(LL_USART_IsActiveFlag_RXNE(UART5)){
      /* RX 버퍼에 데이터가 있으면 아래의 코드 실행 */
      LL_USART_TransmitData8(UART5, LL_USART_ReceiveData8(UART5));//loopback
  }
  /* USER CODE END UART5_IRQn 0 */
  /* USER CODE BEGIN UART5_IRQn 1 */

  /* USER CODE END UART5_IRQn 1 */
}
```

## DMA

만약 DMA로 TX를 사용할 때 보내고자 하는 데이터의 전송완료를 알고싶다면 아래의 코드를 사용하여 complete인터럽트를 활성화 시킨다.
```c
LL_USART_EnableDMAReq_TX(USARTx);
```

DMA로 TX전송중 에러가 발생할 경우를 알고싶다면 아래의 코드를 사용하여 error 인터럽트를 활성화 시킨다.
```c
LL_USART_EnableDMAReq_TX(USARTx);
```

위의 코드를 이용하여 활성화를 시켰다면 아래의 코드를 stm32xxxx_it.c파일에서 
```c
void DMAx_Channelx_IRQHandler(void)
```
위의 형태의 함수중 설정한 DMA번호와 채널번호에 맞는 함수를 아래의 코드처럼 수정한다.
```c
void DMAx_Channelx_IRQHandler(void)
{
  /* USER CODE BEGIN DMAx_Channelx_IRQn 0 */

  if (LL_DMA_IsActiveFlag_TCx(DMAx))
  {
    LL_DMA_ClearFlag_TCx(DMAx);
    /* Call function Transmission complete Callback */
    DMAx_TransmitComplete_Callback();
  }
  else if (LL_DMA_IsActiveFlag_TEx(DMAx))
  {
    /* Call Error function */
    USART_TransferError_Callback();
  }
  /* USER CODE END DMAx_Channelx_IRQn 0 */

  /* USER CODE BEGIN DMAx_Channelx_IRQn 1 */

  /* USER CODE END DMAx_Channelx_IRQn 1 */
}
```


생성된 UART_init()에서 
```
/* USER CODE BEGIN UARTx_Init 1 */

/* USER CODE END UARTx_Init 1 */
```
위의 주석 사이에 아래의 코드를 추가한다.
```c
LL_USART_EnableDMAReq_TX(USARTx);
```
만일 위의 complete와 error interrupt를 사용한다면 아래의 코드도 추가한다.
```c
LL_DMA_EnableIT_TC(DMAx, LL_DMA_CHANNEL_x);
LL_DMA_EnableIT_TE(DMAx, LL_DMA_CHANNEL_x);
```

## 데이터 전송

데이터는 보통 배열이나 구조체를 이용하여 보낸다. 따라서 보내야하는 데이터의 시작주소와 보내는 데이터의 길이를 설정해줘야한다.
```c
LL_DMA_ConfigAddresses(DMAx, LL_DMA_CHANNEL_x,
                         (uint32_t)aTxBuffer,
                         LL_USART_DMA_GetRegAddr(USARTx, LL_USART_DMA_REG_DATA_TRANSMIT),
                         LL_DMA_GetDataTransferDirection(DMAx, LL_DMA_CHANNEL_x));
LL_DMA_SetDataLength(DMAx, LL_DMA_CHANNEL_x, datalength);
```
위와 같이 설정후 아래의 코드를 사용하면 데이터가 전송된다.
```c
LL_DMA_EnableChannel(DMAx, LL_DMA_CHANNEL_x);
```

## Normal Mode vs Circular Mode
normal mode사용시 데이터를 전송하고 싶을때마다 아래의 코드를 사용해야한다.
```c
LL_DMA_EnableChannel(DMAx, LL_DMA_CHANNEL_x);
```

circular mode 사용시 한 번 데이터를 보내기 시작하면 아래의 코드를 사용하기 전까지 계속 반복해서 데이터를 전송한다.
```c
LL_DMA_DisableChannel(DMAx, LL_DMA_CHANNEL_x);
```