## 외부 핀 입력 인터럽트

외부에서 들어오는 디지털입력에 대한 인터럽트 기능이다.
현재 테스트중인 STM32G4계열 MCU는 모든 핀의 입력에 대해 인터럽트를 활성화 시킬 수 있다.

GPIO_EXTI설정을 하고 코드 생성을 진행하면 별다른 활성화 코드없이 바로 사용이 가능하다.

### 인터럽트 발생시

stm32xxxx_it.c 와 비슷한 파일에서 내가 설정한 범위에 있는 EXTI IRQHandler를 찾아서 필요한 코드를 작성하면된다

아래의 예시는 EXTI[15:10]이다.
``` c
void EXTI15_10_IRQHandler(void)
{
  /* USER CODE BEGIN EXTI15_10_IRQn 0 */

  /* USER CODE END EXTI15_10_IRQn 0 */
  if (LL_EXTI_IsActiveFlag_0_31(LL_EXTI_LINE_10) != RESET)
  {
    LL_EXTI_ClearFlag_0_31(LL_EXTI_LINE_10);
    /* USER CODE BEGIN LL_EXTI_LINE_10 */
		
    /* USER CODE END LL_EXTI_LINE_10 */
  }
  if (LL_EXTI_IsActiveFlag_0_31(LL_EXTI_LINE_11) != RESET)
  {
    LL_EXTI_ClearFlag_0_31(LL_EXTI_LINE_11);
    /* USER CODE BEGIN LL_EXTI_LINE_11 */

    /* USER CODE END LL_EXTI_LINE_11 */
  }
  if (LL_EXTI_IsActiveFlag_0_31(LL_EXTI_LINE_12) != RESET)
  {
    LL_EXTI_ClearFlag_0_31(LL_EXTI_LINE_12);
    /* USER CODE BEGIN LL_EXTI_LINE_12 */

    /* USER CODE END LL_EXTI_LINE_12 */
  }
  /* USER CODE BEGIN EXTI15_10_IRQn 1 */

  /* USER CODE END EXTI15_10_IRQn 1 */
}
```