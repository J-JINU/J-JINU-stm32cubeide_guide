정밀한 PWM 혹은 타이머가 필요한 경우 HRTIM모듈을 사용해야한다.

## HRTIM 이란
HRTIM은 High Resolution TIMer의 약자이다.
기존의 타이머로는 부족한 해상도(개인적인 생각으로 MCU clock에 한정된 수준의 PWM신호)를 보완하기 위한 모듈이다.

주된 용도는 높은 Frequency를 가진 매우 정밀한 PWM 신호가 필요한 경우일 것이라고 생각한다. EX) BLDC motor driver, Buck-Boost ...

BLDC driver를 제작하고자 하는 경우에는 필수로 사용해야한다. 일반 타이머는 2채널 PWM출력은 가능하나 역상 PWM이 1khz이하에서 사용가능한 수준이기 때문이다. 그리고 PWM Frequency의 최대치도 낮다.

## HRTIM init
상당히 복잡하다. G474기준으로 설명하자면 HRTIM모듈 내부에 독립적인 6개의 TIMER모듈?이 들어있고 각 모듈은 Compare/Capture Unit을 여러개 가지고 있다. 그리고 각 타이머는 외부출력을 최대 2개까지 가지고 있다.

TIMER를 enable(출력을 내거나 출력없이 사용하거나)한 후 Prescaler와 Period로 Timer의 Frequency를 정한다. Prescaler에 따른 최소/최대 Frequency는 HRTIM cookbook문서를 검색해서 확인할 수 있다.

Compare unit을 여러개 사용가능한데 이를 이용하셔 timer의 1주기 동안 HIGH <-> LOW 신호를 여러번 발생시키는 것도 가능하고 Compare unit과 ADC trigger를 사용하여 PWM의 HIGH 중간에 ADC를 측정하게도 할 수 있다.

자유도가 너무 높고 설정할 수 있는 항목들이 매우 많은 관계로 GUI에서 설정하는 것에서 어려움을 느낄 수 밖에 없으므로 이런저런 설정을 해보면서 익혀나가길 바란다.


## DeadTime 계산하기
Deadtime주파수는 아래와 같다.

fDTG = fHRTIM * X

* fDTG : Deadtime timer count의 주기이다.
* fHRTIM : Clock Configuration에서 HRTIMER의 클럭과 동일하다.

Rising/Falling Value를 크게하면 DT가 늘어난다.
DT계산은 아래와 같다.

DeadTime = tDTG * Rising/Falling Value

* tDTG : 1 / fDTG한 값으로 1주기당 시간이다.

EX) fHRTIM이 170Mhz / fDTG = fHRTIM * 8 / Value = 500인 경우 tDTG는 약 0.73ns가 되고 0.73ns * 500은 365ns가 된다. DeadTime은 계산식 상으로 약 365ns가 된다.
실제로 측정시 약간의 오차는 존재하므로 항상 측정을 하고 적절한 DT를 찾도록 한다.

## HRTIM 실사용
LL_HRTIM_EnableOutput(HRTIM1, LL_HRTIM_OUTPUT_TC1 | LL_HRTIM_OUTPUT_TC2
			| LL_HRTIM_OUTPUT_TD1 | LL_HRTIM_OUTPUT_TD2
			| LL_HRTIM_OUTPUT_TF1 | LL_HRTIM_OUTPUT_TF2);
  LL_HRTIM_TIM_CounterEnable(HRTIM1, LL_HRTIM_TIMER_C | LL_HRTIM_TIMER_D | LL_HRTIM_TIMER_F);