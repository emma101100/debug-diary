# **I2C 中断相关问题**

## **现象**
> **I2C 中断发送或读取数据时触发了 `HardFault_Handler`**

---

## **问题分析**

### **上一次 I2C 发送尚未完成，新的 I2C 发送请求覆盖**

#### **检查点**

✅ **1. 检查 I2C 句柄状态**
- 在 TIM 中断回调中，在调用 `HAL_I2C_Master_Transmit_IT()` 之前，检查 `hi2c->State`：
  ```c
  if (hi2c1.State != HAL_I2C_STATE_READY) {
      printf("I2C is busy, skipping new transmission.\n");
  }
  ```
  - 如果 `State` 不是 `READY`，说明上一次发送仍未完成，不应启动新的 I2C 发送。

✅ **2. 检查 I2C 是否因上一次发送失败进入错误状态**
- 如果 I2C 进入错误状态，它不会自动恢复，需要手动复位：
  ```c
  if (HAL_I2C_GetError(&hi2c1) != HAL_I2C_ERROR_NONE) {
      printf("I2C error detected: 0x%02X\n", HAL_I2C_GetError(&hi2c1));
  }
  ```
  - 如果 `HAL_I2C_GetError()` 返回非 `HAL_I2C_ERROR_NONE`，说明 I2C 可能进入了 `ERROR` 状态。

✅ **3. 确保数据缓冲区 `pData` 是有效的**
- 如果 `HAL_I2C_Master_Transmit_IT()` 访问了一个无效地址（例如 `pData` 被释放或未正确初始化），可能会导致 `HardFault`：
  ```c
  if (data == NULL) {
      printf("I2C TX buffer is NULL!\n");
  }
  ```

✅ **4. 检查是否有中断抢占**
- 在 TIM 和 I2C 中断中调用：
  ```c
  printf("Current ISR: %ld\n", __get_IPSR());
  ```
  - 如果 `__get_IPSR()` 返回的是高优先级的中断（如 UART），说明 I2C 处理中被抢占，可能导致 I2C 发送失败。



---

### **`printf()` 触发 UART 中断，导致 TIM/I2C 任务被抢占**

- `printf()` 会使用 `HAL_UART_Transmit_IT()`，这会触发 UART TX 中断。
- **如果 UART 中断的优先级高于 TIM 或 I2C**，UART 会优先执行，而 I2C 可能会卡在等待中断的状态，导致 I2C 操作失败。

#### **检查点**

✅ **在 TIM 和 I2C 中断里调用：**
```c
printf("%d", __get_PRIMASK());
```
- 观察是否有异常的中断嵌套。

---

### ** 中断嵌套导致栈溢出**

如果 `printf()` 触发了 UART 中断，而 UART 中断比 I2C 或 TIM 的优先级更高，则会导致：
1. **UART 抢占 I2C**，I2C 可能在处理过程中被打断。
2. **I2C 被打断后，可能无法正确执行回调函数**，导致 I2C 传输失败或 HAL 状态机出错。
3. **如果中断嵌套太深，可能导致栈溢出**，最终 MCU 进入 `HardFault_Handler()`。

#### **检查点**
✅ **在 `HardFault_Handler()` 里打印 `MSP` 或 `PSP`，看看是否超出栈范围**。

---

## **解决方案**

### **在 TIM 中断里只设标志位，不直接调用 I2C**

#### **实现步骤**
1. **在 TIM 中断里设标志位**
2. **在主循环（`while(1)`）里轮询标志位，并调用 `HAL_I2C_Master_Receive_IT()`**
3. **等 I2C 完成后，在主循环里 `printf()`**

#### **代码示例**
```c
volatile uint8_t i2c_read_flag = 0;

void TIMx_IRQHandler(void) {
    if (__HAL_TIM_GET_IT_SOURCE(&htim1, TIM_IT_UPDATE) != RESET) {
        __HAL_TIM_CLEAR_IT(&htim1, TIM_IT_UPDATE);
        i2c_read_flag = 1;  // 仅设置标志位，不直接调用 I2C
    }
}

void HAL_I2C_MasterRxCpltCallback(I2C_HandleTypeDef *hi2c) {
    i2c_read_flag = 0;  // 读取完成后清除标志位
}

void main(void) {
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_I2C1_Init();
    MX_TIM1_Init();

    HAL_TIM_Base_Start_IT(&htim1);  // 启动定时器中断
    while (1) {
        if (i2c_read_flag && hi2c1.State == HAL_I2C_STATE_READY) {
        HAL_I2C_Master_Receive_IT(&hi2c1, DEVICE_ADDR, data, len);
            i2c_read_flag = 0;
    }
}
```
---
### 在中断里设置标志位，并在中断结束后触发 I2C 任务

#### 实现步骤

1. 在 TIM 中断里设标志位

2. 在 HAL_I2C_MasterTxCpltCallback() 或 HAL_I2C_MasterRxCpltCallback() 里执行 I2C 任务

```c
volatile uint8_t i2c_busy = 0;  // I2C 传输状态标志
uint8_t tx_data[10] = "Hello";  // 发送的数据

void TIMx_IRQHandler(void) {
    if (__HAL_TIM_GET_IT_SOURCE(&htim1, TIM_IT_UPDATE) != RESET) {
        __HAL_TIM_CLEAR_IT(&htim1, TIM_IT_UPDATE);

        if (!i2c_busy) {  // 仅当 I2C 为空闲时才启动新的 I2C 发送
            i2c_busy = 1;  // 标记 I2C 正在进行
            if (HAL_I2C_Master_Transmit_IT(&hi2c1, DEVICE_ADDR, tx_data, sizeof(tx_data)) != HAL_OK) {
                i2c_busy = 0;  // 如果发送失败，释放标志位
                printf("I2C transmit failed.\n");
            }
        }
    }
}

void HAL_I2C_MasterTxCpltCallback(I2C_HandleTypeDef *hi2c) {
    i2c_busy = 0;  // 释放 I2C 传输锁
    printf("I2C transmit complete.\n");
}

void HAL_I2C_MasterRxCpltCallback(I2C_HandleTypeDef *hi2c) {
    printf("I2C receive complete.\n");
}
```

---

### **使用 DMA**

如果 MCU 支持 `HAL_I2C_Master_Receive_DMA()`，可以让 I2C 使用 DMA，而不是中断模式：

```c
HAL_I2C_Master_Receive_DMA(&hi2c1, DEVICE_ADDR, data, len);
```

---

## **结论**

**问题本质**
 I2C 数据传输是异步操作，如果在上一次传输未完成时就启动新的传输请求，容易破坏 I2C 状态机，从而可能导致 HardFault 或数据丢失。同时，中断嵌套和中断优先级问题（如 UART 中断抢占）也会对传输过程产生干扰。

**综合解决方案**

1. **在中断中仅设置标志位**：

   在 TIM 中断处理函数里，仅设置触发 I2C 操作的标志位，而不直接调用 I2C 发送或接收函数。这有助于避免在中断中执行耗时任务，从而减少中断嵌套和抢占问题。

2. **使用状态变量控制传输**：

   通过全局状态变量（如 `i2c_busy`、`i2c_tx_done`、`i2c_read_flag`）确保当前 I2C 传输完成后再启动下一次传输，防止重复启动引发状态机错误。

3. **在 HAL 回调中处理状态更新**：

   在 I2C 发送或接收完成的回调函数中，及时清除状态变量或设置完成标志，确保传输状态及时更新，为后续操作提供依据。

4. **调整中断优先级**：

   根据系统需求，调整 I2C、TIM 和 UART 等中断的优先级，确保关键的 I2C 中断不被其他中断（如 UART 中断）抢占，从而保证传输的时序和稳定性。

5. **使用 DMA 模式**：

   采用 DMA 方式进行 I2C（和 UART）传输，以降低 CPU 中断负荷，进一步提高传输的稳定性和效率。
