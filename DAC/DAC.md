## OUT MODE
1. Connected to external pin only
    * 외부로 DAC 출력을 내보냄
2. OUTx Connected to on chip-peripherals only
3. connected to external pin and to on chip-peripherials


## Parameter Setting


## 코드 생성 후 추가 작업
DAC 활성화를 위해서 아래의 코드를 추가해야한다
```c 
LL_DAC_Enable(DACx, LL_DAC_CHANNEL_x);
```

이후 값을 변경하고자 한다면 아래의 코드를 이용하여 출력값을 변경시키면된다.
```c
    /* Set the data to be loaded in the data holding register */
    LL_DAC_ConvertData12RightAligned(DACx, LL_DAC_CHANNEL_x, dac_value);
        
    /* Trig DAC conversion by software */
    LL_DAC_TrigSWConversion(DACx, LL_DAC_CHANNEL_x);
```
* 참고사항

  ```c
    LL_DAC_Enable(DACx, LL_DAC_CHANNEL_x | LL_DAC_CHANNEL_y);
  ```
  위의 코드는 불가능하다.
  ```c
    LL_DAC_TrigSWConversion(DACx, LL_DAC_CHANNEL_x | LL_DAC_CHANNEL_y);
  ```
  위의 코드는 가능하다.

## DMA와 TIMER를 이용하여 WAVE 출력하기
1. wave의 주기와 배열의 크기를 기자고 TIMER주기를 설정한다.
2. DAC parameter에서 Trigger를 Timer로 설정한다.
3. 배열에 원하는 WAVE형태의 12bit 데이터를 만든다.
4. 아래의 코드를 이용하여 DMA를 설정한다.
```c
    /* Set DMA transfer addresses of source and destination */
    LL_DMA_ConfigAddresses(DMAx,
                            LL_DMA_CHANNEL_x,
                            (uint32_t)&wave_array,
                            LL_DAC_DMA_GetRegAddr(DACx, LL_DAC_CHANNEL_x, LL_DAC_DMA_REG_DATA_12BITS_RIGHT_ALIGNED),
                            LL_DMA_DIRECTION_MEMORY_TO_PERIPH);

    /* Set DMA transfer size */
    LL_DMA_SetDataLength(DMAx,
                        LL_DMA_CHANNEL_x,
                        WAVEFORM_SAMPLES_SIZE);

    /* Enable DMA transfer interruption: transfer error */
    LL_DMA_EnableIT_TE(DMAx, LL_DMA_CHANNEL_x);

    /* Activation of DMA */
    /* Enable the DMA transfer */
    LL_DMA_EnableChannel(DMAx, LL_DMA_CHANNEL_x);

    /* Enable DAC channel DMA request */
    LL_DAC_EnableDMAReq(DACx, LL_DAC_CHANNEL_x);
```
5. 아래의 코드로 TIMER를 동작시킨다.
```c
    /* Enable counter */
    LL_TIM_EnableCounter(TIMx);
```
6. 아래의 코드로 DAC를 동작시킨다.
```c
    /* Enable DAC channel */
    LL_DAC_Enable(DACx, LL_DAC_CHANNEL_x);
    /* Enable DAC channel trigger */
    LL_DAC_EnableTrigger(DACx, LL_DAC_CHANNEL_x);
```