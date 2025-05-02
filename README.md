# STM32-Smart-Fan-Controller-Auto-Manual-Mode-

// =======================
// File: main.c
// STM32 Smart Fan Controller (Auto/Manual Mode)
// =======================

#include "main.h"
#include "lcd.h"
#include <stdio.h>

// === Global Variables ===
ADC_HandleTypeDef hadc1;
TIM_HandleTypeDef htim2;
UART_HandleTypeDef huart2;  // UART handle

uint8_t mode = 0; // 0: Auto, 1: Manual
uint8_t manual_speed = 50; // % 
uint16_t temperature = 0;

// === Function Prototypes ===
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_ADC1_Init(void);
static void MX_TIM2_Init(void);
static void MX_USART2_UART_Init(void);  // UART init function
void update_LCD_display();
void set_fan_speed(uint8_t speed);
uint16_t read_temperature();
void send_temperature_uart(uint16_t temp);  // Send temperature over UART

// === Main Function ===
int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();
  MX_ADC1_Init();
  MX_TIM2_Init();
  MX_USART2_UART_Init();  // Initialize UART
  LCD_Init();
  HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1);

  LCD_Clear();
  LCD_Set_Cursor(1, 1);
  LCD_Print("Smart Fan Ctrl");
  HAL_Delay(1000);

  while (1)
  {
    temperature = read_temperature();

    // Send temperature data to UART every loop
    send_temperature_uart(temperature);  // Send temperature to UART

   if (mode == 0) // Auto Mode
    {
      uint8_t speed = (temperature > 25) ? (temperature - 25) * 10 : 0;
      if (speed > 100) speed = 100;
      set_fan_speed(speed);
    }
    else // Manual Mode
    {
      set_fan_speed(manual_speed);
    }

   update_LCD_display();
    HAL_Delay(500);
  }
}

// === Temperature Reading Function ===
uint16_t read_temperature()
{
  HAL_ADC_Start(&hadc1);
  HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY);
  uint32_t adc_val = HAL_ADC_GetValue(&hadc1);
  return (adc_val * 330) / 4096; // LM35: 10mV/°C, ADC: 12-bit
}

// === Fan Speed Control Function ===
void set_fan_speed(uint8_t speed)
{
  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, speed);
}

// === LCD Update Function ===
void update_LCD_display()
{
  char line1[16], line2[16];
  snprintf(line1, 16, "Temp: %3dC", temperature);
  snprintf(line2, 16, "%s %3d%%", mode ? "Manual" : "Auto", mode ? manual_speed : (temperature > 25 ? (temperature - 25) * 10 : 0));
  LCD_Set_Cursor(1, 1);
  LCD_Print(line1);
  LCD_Set_Cursor(2, 1);
  LCD_Print(line2);
}

// === UART Function to Send Temperature ===
void send_temperature_uart(uint16_t temp)
{
  char uart_buffer[50];
  snprintf(uart_buffer, sizeof(uart_buffer), "Temperature: %dC\r\n", temp);
  HAL_UART_Transmit(&huart2, (uint8_t *)uart_buffer, strlen(uart_buffer), HAL_MAX_DELAY);
}

// === EXTI Callbacks for Button Press ===
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
  if (GPIO_Pin == GPIO_PIN_0) // Mode Toggle
    mode = !mode;
  else if (GPIO_Pin == GPIO_PIN_1 && mode == 1) // Speed Adjust (Manual Only)
  {
    manual_speed += 10;
    if (manual_speed > 100) manual_speed = 0;
  }
}

// === UART Initialization Function ===
static void MX_USART2_UART_Init(void)
{
  huart2.Instance = USART2;
  huart2.Init.BaudRate = 9600;
  huart2.Init.WordLength = UART_WORDLENGTH_8B;
  huart2.Init.StopBits = UART_STOPBITS_1;
  huart2.Init.Parity = UART_PARITY_NONE;
  huart2.Init.Mode = UART_MODE_TX_RX;
  huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart2.Init.OverSampling = UART_OVERSAMPLING_16;
  if (HAL_UART_Init(&huart2) != HAL_OK)
  {
    Error_Handler();  // Handle UART initialization error
  }
}

