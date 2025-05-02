This project is a Smart Fan Controller system built using the STM32 microcontroller and an LM35 temperature sensor. It dynamically adjusts the fan speed based on the room temperature and allows the user to control the mode (Auto or Manual) and fan speed. The system is designed to operate in two modes:

Auto Mode: The fan speed is automatically adjusted based on the room temperature. As the temperature increases, the fan speed increases proportionally. The fan operates faster as the temperature rises above a threshold (e.g., 25°C).

Manual Mode: The fan speed is controlled manually using push buttons. The user can increase or decrease the fan speed in steps, with a maximum limit of 100%. This mode is useful when the user prefers direct control over the fan speed, regardless of temperature.

Key Features:
Temperature Sensing: The LM35 sensor measures the room temperature. The temperature value is read by the microcontroller through an ADC (Analog to Digital Converter).

PWM Fan Control: The fan speed is controlled using Pulse Width Modulation (PWM) on one of the STM32's timers.

Push Button Control: Two push buttons are used for toggling between Auto and Manual modes and adjusting the fan speed in Manual mode.

LCD Display: A 16x2 LCD is used to display the current temperature, fan speed, and the selected mode (Auto or Manual).

UART Communication: UART is used to communicate with external sensors (if applicable) to transfer data such as temperature readings or other sensor information, allowing for further integration or monitoring.

How It Works:
When the system is powered on, the LCD shows the temperature and the fan mode.

In Auto Mode, the fan speed is adjusted automatically based on the temperature.

In Manual Mode, the user can increase/decrease the fan speed using the buttons.

The fan speed is controlled via PWM, which adjusts the duty cycle to vary the fan speed.

This system can be further enhanced with wireless control or integration with a smart home system.
