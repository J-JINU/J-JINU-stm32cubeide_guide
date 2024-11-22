# stm32cubeide_guide

## 주저리주저리
도무지 STM32cubeide의 설정값에 대한 설명과 데이터시트상의 register가 매치가 안되서 작성함

 해당 문서는 STM32G474RE를 기준으로 제작함 따라서 다른 MCU모델에 동일하게 적용이 되지 않을 수 있음을 알림

 주로 사용하는 기능을 우선적으로 작성하기에 없는 항목이 있을 수 있음
 
 예제 코드를 작성하기 위한 목적이 아니지만 일부 기능 설명에 대한 코드(많은 사람들이 사용하는 HAL library는 배제하고 LL library를 기준으로 작성함)는 작성됨

 Low Layer를 사용하는 이유는 Hal은 코드 양이 너무 많고 api를 보면 쓸데없는 것은 아니지만 현업에서 사용하기에는 불필요한 코드들이 너무 많기 때문이다.
 Hal의 장점으로 이식성에 대한 부분과 사용자 친화적이라는 말을 사용하지만 메모리 많이쓰고 코드양이 많아지고 이로인해 성능이 떨어지는 것이 상당히 마음에 들지 않는다.
 그리고 이식성에 대해서 짚고 가면 현업에서 MCU선정을 하게되면 보통은 1개(main stream) 모델 내지 2개(highend/mainstream/low power) 모델을 선정해서 메모리 사이즈나 핀 갯수정도만 바꿔서 사용하기에 이식성이 좋은 것이 장점이 되는 부분인지는 잘 모르겠다.

 MCU는 기본적으로 성능을 빡빡하게 다 써야한다 라는 것이 개인적인 생각이라 프로토타입을 진짜 엄청나게 빠르게 만들어야하거나 교육용(MCU를 처음 접해본사람)이 아닌 이상 현업 개발자가 HAL을 메인으로 사용한다는 것은 문제가 있다고 생각한다.
 차라리 LL을 사용하고 성능 어올려서 다른 연산처리를 더 한다던지 RTOS를 도입하고 RTOS에서 발생하는 스케줄링이나 컨텍스트 스위칭 비용을 조금이나마 충당하는 것이 더 이득이라고 생각한다.

 LL라이브러리의 가장 큰 문제는 cubeMX에서 enable 시켜놔도 실제로 동작하지 않는 것을 경험 할 수 있다는 것이다.

이렇게 해놓은 이유는 아마도 잘못된 설정으로 인한 MCU 고장을 방지하기 위해서가 아닐까 싶다. 하지만 사용자 입장에서는 기능을 enable시켜놓았는데 동작을 안해서 짜증나는 경우가 발생한다.

따라서 현업에서는 자주쓰는 기능들을 최대한으로 활성화 해두고 초기설절 함수들도 작성한 뒤에 copy해서 사용하거나 기능별로 init함수를 따로 정리를 해두고 프로젝트 상황에 맞게 가지고 와서 사용하는 것이 좋아보인다.

 
## 목차
1. [SYSTICK](SYSTICK/SYSTICK.md)
2. [GPIO](GPIO/GPIO.md)
3. [TIMER](TIMER/TIMER.md)
4. [ADC](ADC/ADC.md)
5. [UART](UART/UART.md)
6. [SPI](SPI/SPI.md)
7. [DMA](DMA/DMA.md)
8. [NVIC](NVIC/NVIC.md)
9. [EXTI](EXTI/EXTI.md)