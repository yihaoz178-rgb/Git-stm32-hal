# UART 配置（STM32F1 HAL）

> 以下 HAL 函数声明在 `stm32f1xx_hal_uart.h`，实现主要在 `stm32f1xx_hal_uart.c`。示例使用 USART1 默认引脚 PA9（TX）、PA10（RX），8 位数据、无校验、1 位停止位。
>
> 中断接收流程：`HAL_UART_Receive_IT()` → `USART1_IRQHandler()` → `HAL_UART_IRQHandler()` → `HAL_UART_RxCpltCallback()`。

***

### **HAL_UART_Init()**

1. **作用：** 根据句柄中的 `Init` 配置 UART 通信参数并使能外设；首次初始化时会调用 `HAL_UART_MspInit()` 配置底层硬件。
2. **入参：** `UART_HandleTypeDef *huart`，UART 句柄地址。
3. **主要配置：**
   - `Instance`：串口实例，如 `USART1`。
   - `BaudRate`：波特率，如 `115200`。
   - `WordLength`：帧数据位数，如 `UART_WORDLENGTH_8B`。
   - `StopBits`：停止位，如 `UART_STOPBITS_1`。
   - `Parity`：校验位，如 `UART_PARITY_NONE`。
   - `Mode`：收发模式，如 `UART_MODE_TX_RX`。
   - `HwFlowCtl`：硬件流控，如 `UART_HWCONTROL_NONE`。
   - `OverSampling`：过采样，如 `UART_OVERSAMPLING_16`；普通 STM32F1 示例使用 16 倍过采样。
4. **回参：** `HAL_OK` 表示初始化成功；失败时可返回 `HAL_ERROR` 等 HAL 状态。
5. **代码示例：**

   ```c
   UART_HandleTypeDef huart1 = {0};  // 保持句柄在收发和中断期间有效

   void USART1_Init(void)
   {
       huart1.Instance = USART1;
       huart1.Init.BaudRate = 115200;
       huart1.Init.WordLength = UART_WORDLENGTH_8B;
       huart1.Init.StopBits = UART_STOPBITS_1;
       huart1.Init.Parity = UART_PARITY_NONE;
       huart1.Init.Mode = UART_MODE_TX_RX;
       huart1.Init.HwFlowCtl = UART_HWCONTROL_NONE;
       huart1.Init.OverSampling = UART_OVERSAMPLING_16;

       if (HAL_UART_Init(&huart1) != HAL_OK)
       {
           Error_Handler();
       }
   }
   ```

***

### **HAL_UART_MspInit()**

1. **作用：** 配置 UART 使用的时钟、GPIO；使用中断收发时还需配置 NVIC。HAL 提供弱定义，通常在用户文件中实现同名函数，由 `HAL_UART_Init()` 调用。
2. **入参：** `UART_HandleTypeDef *huart`，通过 `huart->Instance` 判断要配置的串口。
3. **代码示例：**

   ```c
   void HAL_UART_MspInit(UART_HandleTypeDef *huart)
   {
       GPIO_InitTypeDef GPIO_InitStruct = {0};

       if (huart->Instance == USART1)
       {
           __HAL_RCC_USART1_CLK_ENABLE();
           __HAL_RCC_GPIOA_CLK_ENABLE();

           // USART1_TX：PA9，复用推挽输出
           GPIO_InitStruct.Pin = GPIO_PIN_9;
           GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
           GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
           HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

           // USART1_RX：PA10，输入模式
           GPIO_InitStruct.Pin = GPIO_PIN_10;
           GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
           GPIO_InitStruct.Pull = GPIO_NOPULL;
           HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

           HAL_NVIC_SetPriority(USART1_IRQn, 1, 0);
           HAL_NVIC_EnableIRQ(USART1_IRQn);
       }
   }
   ```

***

### **USART1_IRQHandler(void)**

1. **作用：** USART1 的中断服务函数。CPU 进入该函数后，交给 HAL 中断处理函数识别收发或错误事件。
2. **入参：** 无。
3. **代码示例：** 通常放在 `stm32f1xx_it.c`；`huart1` 须为前面初始化的同一个句柄。

   ```c
    /* UART1回调函数 */
    void USART1_IRQHandler(void)
    {
	    HAL_UART_IRQHandler(&huart1);
	    HAL_UART_Receive_IT(&&huart1,(uint8_t *)G_rx_buffer,1);	/* 开启USART1中断接收 1字符 */
    }
   ```

   > 仅使能 NVIC 还不够：要先调用 `HAL_UART_Receive_IT()` 或 `HAL_UART_Transmit_IT()` 启动相应的中断收发。

***

### **HAL_UART_RxCpltCallback()**

1. **作用：** 中断方式接收完指定的 `Size` 个数据后，由 HAL 调用。此时本次接收已经结束；需要连续接收时，应再次启动 `HAL_UART_Receive_IT()`。
2. **入参：** `UART_HandleTypeDef *huart`，发生接收完成事件的 UART 句柄。
3. **代码示例：** 接收缓冲区要在中断返回后仍然有效，因此使用静态或全局变量。

   ```c
   static uint8_t rx_byte;

   void USART1_StartReceive(void)
   {
       // 初始化 USART1 后调用一次，开始接收第一个字节
       HAL_UART_Receive_IT(&huart1, &rx_byte, 1);
   }

   void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
   {
       if (huart->Instance == USART1)
       {
           // 此处处理 rx_byte；中断中尽量避免耗时操作
           HAL_UART_Receive_IT(huart, &rx_byte, 1);  // 继续接收下一个字节
       }
   }
   ```

***

### **HAL_UART_Transmit()**

1. **作用：** 阻塞方式发送指定长度的数据；函数会等待发送过程完成或超时。
2. **入参：** `huart` 为 UART 句柄；`pData` 为发送数据地址；`Size` 为发送字节数；`Timeout` 为超时时间（毫秒，可用 `HAL_MAX_DELAY`）。
3. **回参：** `HAL_OK`、`HAL_ERROR`、`HAL_BUSY` 或 `HAL_TIMEOUT`。
4. **代码示例：**

   ```c
   const uint8_t message[] = "Hello\r\n";
    HAL_UART_Transmit(&huart1, message, sizeof(message) - 1, 1000);
   ```

***

### **HAL_UART_Transmit_IT()**

1. **作用：** 启动中断方式发送，调用后立即返回；全部数据发送完成时由 HAL 调用 `HAL_UART_TxCpltCallback()`。
2. **入参：** `huart` 为 UART 句柄；`pData` 为发送数据地址；`Size` 为发送字节数。
3. **回参：** `HAL_OK` 表示成功启动，**不代表已经发完**；也可能返回 `HAL_ERROR` 或 `HAL_BUSY`。
4. **代码示例：** 发送完成前，缓冲区必须保持有效且不要修改。

   ```c
   static const uint8_t message[] = "Hello\r\n";
   HAL_UART_Transmit_IT(&huart1, message, sizeof(message) - 1U);
   ```

***

### **HAL_UART_Receive()**

1. **作用：** 阻塞方式接收指定长度的数据；收满 `Size` 个数据或超时后返回。
2. **入参：** `huart` 为 UART 句柄；`pData` 为接收缓冲区地址；`Size` 为接收字节数；`Timeout` 为超时时间（毫秒）。
3. **回参：** `HAL_OK`、`HAL_ERROR`、`HAL_BUSY` 或 `HAL_TIMEOUT`。
4. **代码示例：**

   ```c
   uint8_t data[4] = {0};
   HAL_UART_Receive(&huart1, data, sizeof(data), 1000U);
   ```

***

### **HAL_UART_Receive_IT()**

1. **作用：** 启动中断方式接收，调用后立即返回；收满 `Size` 个数据后执行 `HAL_UART_RxCpltCallback()`。
2. **入参：** `huart` 为 UART 句柄；`pData` 为接收缓冲区地址；`Size` 为需要接收的数据个数。
3. **回参：** `HAL_OK` 表示成功启动，**不代表已收到数据**；也可能返回 `HAL_ERROR` 或 `HAL_BUSY`。
4. **代码示例：** 接收缓冲区在回调触发前必须保持有效。

   ```c
   static uint8_t rx_buffer[8];
   HAL_UART_Receive_IT(&huart1, rx_buffer, sizeof(rx_buffer));
   // 接收满 8 个字节后，HAL 才会调用 HAL_UART_RxCpltCallback()
   ```

***

> **注意：** 上述字节缓冲区示例采用 `UART_WORDLENGTH_8B` 且无校验。若改为 9 位数据且无校验，收发缓冲区需要按 `uint16_t` 数据单元准备，`Size` 也按数据单元计数。

> 参考：[ST 官方 STM32F1 UART HAL 头文件](https://github.com/STMicroelectronics/stm32f1xx-hal-driver/blob/master/Inc/stm32f1xx_hal_uart.h)、[实现文件](https://github.com/STMicroelectronics/stm32f1xx-hal-driver/blob/master/Src/stm32f1xx_hal_uart.c)。
