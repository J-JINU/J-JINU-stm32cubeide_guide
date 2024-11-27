## TIMER set period
타이머의 주기 T 계산

T = ClockSource / (Prescaler + 1) / CounterPeriod

주기를 계산해서 값을 넣는다.

타이머 주기변경은 아래의 함수를 사용한다.
```c
LL_TIM_SetAutoReload(TIM_TypeDef *TIMx, uint32_t AutoReload)
LL_TIM_SetPrescaler(TIM_TypeDef *TIMx, uint32_t Prescaler)
```

## TIMER interrupt
NVIC탭에서 interrupt enable한다.

코드 생성후
```c
void MX_TIMx_Init(void)
```
위 함수의 마지막 "USER CODE"작성 부분에 아래의 형태로 코드를 추가한다.

```c
LL_TIM_EnableIT_UPDATE(TIMx);
LL_TIM_EnableCounter(TIMx);
```

## PWM
주기 설정 이후 "Automatic Output State"를 enable로 변경한다.

코드 생성후
```c
void MX_TIMx_Init(void)
```
위 함수의 마지막 "USER CODE"작성 부분에 아래의 형태로 코드를 추가한다.

```c
LL_TIM_CC_EnableChannel(TIMx, LL_TIM_CHANNEL_CHx | LL_TIM_CHANNEL_CHxN);
LL_TIM_EnableCounter(TIMx);
LL_TIM_OC_SetCompareCHx(TIMx, duty);
```
이후 듀티 조정은 
```c
LL_TIM_OC_SetCompareCHx(TIMx, duty)
```
함수로 할 수 있다.

+ 반전된 PWM신호는 slew가 심해서 1kHz이하에서나 사용이 가능할 것을 보임