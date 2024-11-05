# Stepper Motor ULN2003APG control with STM32 Microcontroller 
Below is an easy and short code to control a ULN2003APG Stepper Motor with an STM32 microcontroller using the Half Drive method.
The full tutorial video is available on YouTube: https://www.youtube.com/watch?v=bE-lD6TE_EM

    /* USER CODE BEGIN 0 */
    //Setting up the Microcontroller pins (STM32 L432KC pins A1,A2,A3, and A4)
    #define IN1_PIN GPIO_PIN_1
    #define IN1_PORT GPIOA
    #define IN2_PIN GPIO_PIN_3
    #define IN2_PORT GPIOA
    #define IN3_PIN GPIO_PIN_4
    #define IN3_PORT GPIOA
    #define IN4_PIN GPIO_PIN_5
    #define IN4_PORT GPIOA
    
    // Step sequence for motor control (half-drive)
    const uint8_t step_sequence[8][4] = {
        {1, 0, 0, 0},
        {1, 1, 0, 0},
        {0, 1, 0, 0},
        {0, 1, 1, 0},
        {0, 0, 1, 0},
        {0, 0, 1, 1},
        {0, 0, 0, 1},
        {1, 0, 0, 1}
    };
    
    // Function for a microsecond delay using Timer 1
    //With the microcontroller running at 80Mhz, the TIM1 needs to be initialized..
    //..with a prescaler of 79 (80-1) and the default maximum counter period of 65535..
    //.. so that it can run at 1MHz for us to get the 1 microsecond delay
    void microDelay(uint16_t delay) {
        __HAL_TIM_SET_COUNTER(&htim1, 0);
        while (__HAL_TIM_GET_COUNTER(&htim1) < delay);
    }
    
    // Function to set GPIO pins according to step sequence
    void setPins(uint8_t step[4]) {
        HAL_GPIO_WritePin(IN1_PORT, IN1_PIN, step[0] ? GPIO_PIN_SET : GPIO_PIN_RESET);
        HAL_GPIO_WritePin(IN2_PORT, IN2_PIN, step[1] ? GPIO_PIN_SET : GPIO_PIN_RESET);
        HAL_GPIO_WritePin(IN3_PORT, IN3_PIN, step[2] ? GPIO_PIN_SET : GPIO_PIN_RESET);
        HAL_GPIO_WritePin(IN4_PORT, IN4_PIN, step[3] ? GPIO_PIN_SET : GPIO_PIN_RESET);
    }
    
    // Function to move the motor in either clockwise or counterclockwise direction
    void stepMotor(int steps, uint16_t delay, int direction) {
        int seq_len = sizeof(step_sequence) / sizeof(step_sequence[0]);
        for (int x = 0; x < steps; x++) {
            int step_index = (direction == 1) ? x % seq_len : (seq_len - (x % seq_len)) % seq_len;
            setPins((uint8_t*)step_sequence[step_index]);
            microDelay(delay);
        }
    }
    /* USER CODE END 0 */


    /* USER CODE BEGIN 2 */
    HAL_TIM_Base_Start(&htim1);  // Start Timer 1
    /* USER CODE END 2 */


    //MAIN WHILE LOOP - Controlling the motor how we want
    /* USER CODE BEGIN 3 */
    //It takes the motor 4096 steps to do 1 full revolution
    //The 1000 is the delay in between the steps to control the speed
    //The 1 is for ClockWise rotation (-1 for CounterClockWise)
	  stepMotor(4096, 1000, 1);  
	  HAL_Delay(2000);
	  stepMotor(2048, 1000, -1);  
	  HAL_Delay(2000);
    }
    /* USER CODE END 3 */
      

    
