ADC는 다양한 기능을 가지고 있어서 parameter를 이해하는 것이 난해할 수 있다.

## 기본 용어 설명
 - 아래의 명명 규칙은 개인적으로 사용하는 명칭이다.
1. ADCx라고 있는 것은 앞으로 "ADCx 모듈"이라고 명명한다.
2. ADCx안에 INx는 "채널"이라고 명명한다.

각 모듈별로 같은 번호의 채널이라도 항상 같은 pin이 아니고 다른 pin일 수도 있다.

일부MCU는 ADC모듈 2개를 하나로 합쳐서 MASTER/SLAVE 연결을 해둔 것도 있다. G4모델은 해당 형태로 존재한다.

## Channel 선택
필요한 ADC port 갯수만큼 선택하면된다. 차동모드도 지원하기 때문에 필요하다면 사용하라
여러개의 channel을 선택할 경우 선택된 모든 채널은 Sequence에 포함되게 된다.

## MODE
1. Independent
2. Dual combined regular simultaneous + injected simultaneous
3. Dual regular simultaneous + alternate trigger
4. Dual Interleaved mode + injected simultaneous mode
5. Dual injected simultaneous only
6. Dual regular simultaneous only
7. Dual interleaved only
8. Dual alternate trigger

## Clock Prescaler
묶여있는 모듈은 clock을 공유한다.
1. Synchronous
    * SYSCLK를 ADC clock으로 사용
2. Asynchronous
    * PLLP를 ADC clock으로 사용

## Scan Conversion Mode
하나의 채널의 샘플링이 끝나면 다음 트리거에서 sequence에 포함된 다음 채널의 샘플링을 진행한다.

## End Of Conversion
1. End of Single(EOC flag) 단일 채널 끝나면 인터럽트 발생시킬지
2. End of Sequence(EOS flag)시퀀스 끝나면 인터럽트 발생시킬지


## ADC start trigger
1. ADC_start함수호출
2. 외부 핀 인터럽트로 시작
3. 타이머 인터럽트로 시작