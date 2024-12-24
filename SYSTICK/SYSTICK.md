```c
LL_SYSTICK_EnableIT();
```
systick 인터럽트를 발생시키기위해서 위의 함수를 main의 while이전에 작성해야 1ms인터럽트가 동작한다.

```c
void SysTick_Handler(void)
```
systick이 1ms마다 인터럽트 발생시켜서 위의 함수를 호출함

프로그램 루프가 시작하기전(또는 각 peripheral들이 활성화 되기전)에 1~2ms정도 딜레이를 주어 클럭이 안정화되는 것을 기다리는 것이 좋다.