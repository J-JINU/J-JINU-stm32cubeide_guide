## SPI 통신이란
SPI 통신은 동기식 통신으로 비동기 serial통신과 다르게 start stop bit가 필요하지 않다. 따라서 다른 비동기 통신과 다르게 통신속도의 합의가 굳이 필요하지 않으며 SPI통신을 사용하려고 하는 모듈에서 통신 CLOCK을 받아줄수만 있다면 어디든 사용이 가능하다. 배선은 주로 4선식 또는 3선식을 사용하며 4선식의 비중이 높다고 생각된다.
### SPI 통신의 장점
다른 통신방식들(uart, i2c, can)보다 고속으로 데이터를 주고 받을 수 있는 장점이 있다.

또한 최근에는 QSPI라고하여 클럭 1회당 4비트까지 전송할 수 있도록 데이터 버스용 배선을 추가하여 더 높은 전송률을 가진 모듈들도 많이 출시되고 있다.

Master에서 여러개의 ChipSelect를 사용하여 Slave들을 제어할 수 있다.

### SPI 통신의 단점
4선식의 경우 클럭, Data출력, Data입력, CS로 총 4개의 배선이 필요하다. 하지만 SPI통신을 해야하는 Slave의 갯수만큼 CS핀이 증가하므로 하드웨어적으로 PIN갯수가 모자란 경우에는 여러개의 Slave를 제어하는데 어려움이 발생할 수도 있다.

높은 전송률에 의하여 실질적으로 전선을 이용한 배선을 가지고 하드웨어 구성하는 것에는 한계가 너무 명확하여 PCB상에서만 하드웨어 구성을 할 수 밖에 없는 제약조건이 존재한다.(통신속도와 통신거리는 반비례한다)

### SPI 총평
Can통신이나 이더넷과는 다르게 제어해야하는 장치가 늘어날 수록 배선수가 늘어나는 것과 Master가 정해져야한다는 것이 단점이지만 Can/Ethernet과는 다르게 구현비용이 작으면서 높은 데이터 전송률을 가지고 있다는 점에서 매우 유용하다고 생각이 되고 이 점은 다른 Serial통신(RS232, RS485, I2C)들과 차별점이라고 생각된다.


## UI에서 setup
* Data Size는 일반적으로 8bit를 사용한다.
* 통신속도는 연결 대상에 따라 변경된다.
* Clock Polarity/Clock Phase는 연결대상에 따라 변경된다.

## initialize
* 통신을 활성화 하려면 아래의 코드를 사용해야한다.
```c
LL_SPI_Enable(SPIx);
```

* 만약 통신 속도를 바꾸려면 BaudRate의 Prescaler를 조정하여 적정한 통신속도로 설정하는 것이 편하다.
* 왜냐하면 Clock까지 실시간으로 변경시 SPI에서 사용하는 Clock을 공유하는 다른 Peripheral에 영향을 미칠수도 있기 때문이다
* 아래의 코드를 사용하여 통신속도를 조정한다.
```c
LL_SPI_SetBaudRatePrescaler(SPIx, BaudRate);
```
BaudRate위치에는 LL_SPI_BAUDRATEPRESCALER_DIVxxx로 #define되어있으므로 사용하거나 직접 숫자를 입력해서 사용하면된다.

* 장치마다 SPI MODE는 다른 경우가 있다.
* SPI MODE설정은 아래의 코드를 이용하여 설정한다.
``` c
  LL_SPI_SetClockPolarity(SPIx, ClockPolarity);
  LL_SPI_SetClockPhase(SPIx, ClockPhase);
```

* First Bit도 모듈에 따라 알아서 설정해서 사용하면된다.
* 함수는 생략한다.

## interrupt
일반적으로 4선식 SPI통신은 출력과 입력이 동시에 발생한다.
하지만 Memory모듈의 경우 입력 커맨드와 출력데이터가 다른 경우가 있으므로 알아서 잘 설정하기 바란다.

개인적으로 SPI사용시 입출력이 동시에 발생하므로 interrupt는 입력에만 사용하는 편이다.
따라서 아래의 코드를 입력하여 입력 interrupt를 활성화 한다.
``` c
LL_SPI_EnableIT_RXNE(SPIx);
```

SPI통신사용시 에러가 발생할 경우를 대비하려면 아래의 interrupt활성화 코드를 입력한다.
``` c
LL_SPI_EnableIT_ERR(SPIx);
```

참고로 출력 interrupt활성화 코드는 아래와 같다.
``` c
LL_SPI_EnableIT_TXE(SPIx);
```

## 입출력

입력과 출력을 interrupt를 사용하지 않고 바로 확인해볼 수 있는 함수는 아래와 같다.
``` c
uint8_t SPIx_TransmitReceive(uint8_t data){
  while(LL_SPI_IsActiveFlag_BSY(SPIx));
  while(!LL_SPI_IsActiveFlag_TXE(SPIx));
  LL_SPI_TransmitData8(SPIx, data);

  while(!LL_SPI_IsActiveFlag_RXNE(SPIx));
  return LL_SPI_ReceiveData8(SPIx);
}
```

만일 interrupt를 사용하는 경우에는 아래의 코드를 제거한다.
``` c
  while(!LL_SPI_IsActiveFlag_RXNE(SPIx));
  return LL_SPI_ReceiveData8(SPIx);
```
그리고 stm32g4xx_it.c파일에 SPIx_IRQHandler함수가 있을 것이다. 아니면 비슷한 이름의 함수가 있을 것이다. 이 함수에 아래의 코드를 추가하여 입력데이터를 읽도록 한다.
``` c
  if(LL_SPI_IsActiveFlag_RXNE(SPIx)){
	  LL_SPI_ReceiveData8(SPIx);
  }
```

## DMA사용
위의 입출력 항목에서 첫번쨰 코드를 이용하여 SPI통신을 하게되면 데이터의 양이 많을 수록 MCU는 아무런 작업을 하지 못하는 시간이 늘어나게 된다.

따라서 interrupt기반으로 SPI통신을 하거나 DMA를 사용하여 SPI통신을 해야 MCU의 성능을 최대한으로 뽑아낼 수 있다.
당연히 DMA를 사용하는 것이 interrupt기반보다 더 성능향상이 뚜렷하다. SPI통신속도보다 MCU에서 명령어 처리해서 DMA버퍼를 설정하는 것이 훨씬 속도가 빠르기 때문에 2바이트 이상이라면 DMA를 사용하는 것을 추천한다. 

다만 DMA로 처리하는 경우 tx와 rx의 버퍼를 동일한 크기로 2개를 사용해야하기 때문에 메모리를 2배로 사용하게 된다는 단점이 있다.

DMA를 사용하기 위해서는 아래의 코드를 사용하여 DMA를 활성화 한다.
``` c 
 LL_SPI_EnableDMAReq_TX(SPI1);
 LL_SPI_EnableDMAReq_RX(SPI1);
```

데이터를 송수신하기위해서는 아래의 코드를 이용하여 송수신을 진행한다.

``` c
	//dma spi tx set
	LL_DMA_ConfigAddresses(DMAx, LL_DMA_CHANNEL_x, (uint32_t) tx_buf,
			LL_SPI_DMA_GetRegAddr(SPIx),
			LL_DMA_GetDataTransferDirection(DMAx, LL_DMA_CHANNEL_x));
	LL_DMA_SetDataLength(DMA1, LL_DMA_CHANNEL_x, Length);

	//dma spi rx set
	LL_DMA_ConfigAddresses(DMAx, LL_DMA_CHANNEL_x, LL_SPI_DMA_GetRegAddr(SPIx),
			(uint32_t) rx_buf,
			LL_DMA_GetDataTransferDirection(DMAx, LL_DMA_CHANNEL_x));
	LL_DMA_SetDataLength(DMAx, LL_DMA_CHANNEL_x, Length);

	LL_DMA_EnableChannel(DMAx, LL_DMA_CHANNEL_x);
	LL_DMA_EnableChannel(DMAx, LL_DMA_CHANNEL_x);
```