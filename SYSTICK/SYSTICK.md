```c
LL_SYSTICK_EnableIT();
```
systick 인터럽트를 발생시키기위해서 위의 함수를 main의 while이전에 작성해야 1ms인터럽트가 동작한다.

```c
void SysTick_Handler(void)
```
systick이 1ms마다 인터럽트 발생시켜서 위의 함수를 호출함