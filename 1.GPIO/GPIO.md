# GPIO配置
>以下函数调用所在文件位置:
>stm32f1xx_hal_gpio.c
***

### **__HAL_RCC_GPIOx_CLK_ENABLE()**
  1. **作用：开启GPIO端口时钟**
  2. **入参：** 无
  3. **主要寄存器**
        - F1 ： RCC_APB2ENR
        - F2 ： RCC_APB1ENR
        - F3 ： RCC_APB1ENR
        - F4 ： RCC_AHB4ENR
  3. **代码示例**
   ``` 
	__HAL_RCC_GPIOB_CLK_ENABLE()  //设置GPIOB端口的时钟
   ``` 
*** 
### **HAL_GPIO_Init()**
1. **作用：初始化GPIO端口模式**
2. **入参:**
   1. GPIOx : GPIOA / GPIOB ~  GPIOG
   2. &GPIO_InitStructure : 结构体地址   
3. **结构体配置**
   - 定义结构体：GPIO_InitTypeDef
   - 结构体命名: GPIO_InitStructure（通用的命名） 
   - **.Mode** —(模式) ：
      - GPIO_MODE_ANALOG ——————模拟模式
      - GPIO_MODE_INPUT —浮空输入
      - GPIO_MODE_INPUT ——————普通输入模式
      - GPIO_MODE_OUTPUT_OD————开漏输出 
      - GPIO_MODE_OUTPUT_PP————推挽输出 
      - GPIO_MODE_AF_OD————复用开漏输出 
      - GPIO_MODE_AF_PP————复用推挽输出
      - GPIO_MODE_AF_INPUT————复用功能输入 （等价于普通输入） 
        >推挽输出 ：可以直接输出稳定高低电平，进行驱动外部设备，但不支持线与，容易造成GPIO引脚短路，毁坏芯片。
        >开漏输出 ：只能输出低电平，输出高电平需要外接上拉电阻，支持线与，常用于IIC通信等功能。
        >上拉输入 ：引脚内部为高电平，抗干扰性好。
        >下拉输入 ：引脚内部为低电平，抗干扰性好。
        >浮空输入 ：引脚为高阻态等效为开路，易受干扰
   - **.Pin** —(端口) ：
     - GPIO_Pin_0 ~ GPIO_Pin_15 
     - GPIO_Pin_All ——(选中所有端口)
    - **.Pull** —(内部电平) ：
        - GPIO_NOPULL———无上拉、无下拉
        - GPIO_PULLUP ———启用内部上拉电阻
        - GPIO_PULLDOWN——启用内部下拉电阻
            >在F1芯片，IO口配置为输出模式时禁止使用内部上下拉，设置时为无效
   - **.Speed** —(速率) ：
     - GPIO_SPEED_FREQ_LOW ——2兆赫兹 
     - GPIO_SPEED_FREQ_MEDIUM ——10兆赫兹 
     - GPIO_SPEED_FREQ_HIGH ——50兆赫兹
     
4. **代码示例** 
   - 将GPIOA端口的Pin7引脚初始化为：推挽输出模式，以10兆赫兹运行
    ```
    /*代码部分*/

	GPIO_InitTypeDef GPIO_InitStructure = {0};        	   //定义GPIO初始化结构体

	GPIO_InitStructure.Mode 	= GPIO_MODE_OUTPUT_PP;     //设置为推挽输出
	GPIO_InitStructure.Pin 	= GPIO_PIN_7 ; 	               //初始化引脚
	LGPIO_InitStructure.Pull 	= GPIO_NOPULL ;			   //不上拉，也不下拉
	GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_LOW;		   //设置为低速模式	
	HAL_GPIO_Init(GPIOB,&GPIO_InitStructure);			   //初始化
   ```  
***

### **HAL_GPIO_WritePin()**
1. **作用：** 配置GPIO引脚电平
2. **入参：** 
     1. GPIOA~G
     2. GPIO_Pin_0~15  / GPIO_PIN_All 
     3. GPIO_PIN_RESET (低电平)/ GPIO_PIN_SET (高点平)
3. **代码示例**
    ```
    //设置GPIO的PA1引脚为低电平
	HAL_GPIO_WritePin(GPIOA,GPIO_PIN_1,GPIO_PIN_RESET);
    ```
***  

### **HAL_GPIO_TogglePin()**
1. **作用：** 翻转GPIO引脚电平
2. **入参：** 
     - GPIOA~G
     - GPIO_Pin_0~15  / GPIO_PIN_All 
3. **代码示例**
    ```
    //翻转IO口PA1引脚电平
	HAL_GPIO_TogglePin(GPIOA,GPIO_PIN_1);
    ```
***  

### **HAL_GPIO_ReadPin()**
1. **作用：** 读取GPIO引脚电平
2. **入参：** 
     - GPIOA~G
     - GPIO_Pin_0~15  / GPIO_PIN_All 
3. **回参：**
     - GPIO_PIN_RESET ———低电平
     - GPIO_PIN_SET  ————高电平 
3. **代码示例**
    ```
    //读取IO口PA1引脚是否是低电平，并判断
	if(HAL_GPIO_ReadPin(GPIOA,GPIO_PIN_1) == GPIO_PIN_RESET)
   {
      /*为低电平*/
   }
   else
   {
      /*为高电平*/
   }
    ```
***  


