# PrecisionTemp Controller

## Overview
**PrecisionTemp Controller** is an MCU-based heating system developed using **STM32F103** microcontrollers. It implements a **PID control algorithm** to maintain a target temperature, allowing users to set and monitor temperature via **UART communication** with a PC.  

Temperature is measured using a **MAX6675 K-type thermocouple interface**, and power is delivered to a heater (represented by a light bulb) using a **TRIAC**, controlled by the MCU based on PID calculations.

This project can be applied to **ovens, heaters, and temperature-critical embedded applications**.

---

## Features
- PID-based temperature control  
- Real-time temperature monitoring via UART  
- TRIAC-based AC power control  
- SPI communication with MAX6675 thermocouple module  
- Zero-cross detection circuit for precise AC phase control  

---

## Hardware Components
- STM32F103C8T6 (Programmer: ST-LINK V2)  
- STM32F103 V2 SMART (MCU)  
- TRIAC (for AC power control)  
- Type K Thermocouple  
- MAX6675 Thermocouple-to-Digital Converter  
- 9V Transformer  
- Opto-isolators: 4N25 (zero-cross protection), 4N32 (TRIAC trigger protection)  
- Comparator: LM311N  

---

## Circuit Diagram
<img width="1856" height="594" alt="Screenshot 2025-10-05 201806" src="https://github.com/user-attachments/assets/4b8c5931-74cc-461d-9da1-a2771c65fe6e" />

*Figure 1: STM32CubeMX pin configuration and whole setup*
### Zero-Crossing Circuit
The zero-crossing circuit outputs a rising/falling edge whenever the AC input crosses 0V. It uses a **comparator (LM311N)** and **opto-coupler (4N25)** for isolation.

<img width="871" height="566" alt="Screenshot 2025-10-05 185638" src="https://github.com/user-attachments/assets/14120671-5f2a-45cf-a1e7-1efec7600cc6" />
*Figure 2: Zero-crossing detection circuit*

**Output Example:**  
- Rising/falling edge occurs ~96.7 μs after zero-cross detection  
- Voltage swings from 0–3.3V for MCU logic input  
<img width="641" height="503" alt="Screenshot 2025-10-05 203224" src="https://github.com/user-attachments/assets/a9fd6be6-1490-4f0b-877e-d1e9efed990c" />


*Figure 3: Input AC vs zero-cross output signal*

---

### TRIAC Triggering via Timer
The MCU uses **Timer 2 output compare** to trigger the TRIAC based on zero-cross detection.  
- Prescaler: 36 
- Timer clock: 2 MHz (0.5 µs per tick)  
- Full sine wave: 40,000 ticks (50 Hz)
 <img width="529" height="473" alt="Screenshot 2025-10-05 203655" src="https://github.com/user-attachments/assets/07810583-d686-4f05-a341-bc487ef99682" />

*Figure 4: Output compare mode on Timer 2*

#### Key Functions
```c
void update_CCR3(uint32_t CCR1);
void TriacTriggerCallback(CCRxData *ccr3Data);
void config_pulse(CCRxData *ccr3Data, uint32_t firingAngle, uint32_t width);
HAL_TIM_OC_Start_IT(&htim2,TIM_CHANNEL_1);

##update_CCR3: Updates timer CCR register for next pulse
##TriacTriggerCallback: Interrupt triggered at CNT==0 and CNT==CCR3
##config_pulse: Calculates firing angle and pulse width
##HAL_TIM_OC_Start_IT: Enables timer interrupt

```
###TRIAC Triggering Circuit
-<img width="553" height="483" alt="Screenshot 2025-10-05 204422" src="https://github.com/user-attachments/assets/fd66f3c3-d47f-43c7-9ca3-fbb66ad66a6c" />

-<img width="549" height="516" alt="Screenshot 2025-10-05 204501" src="https://github.com/user-attachments/assets/3908bf39-cb1a-4845-ac35-4a024d3552b1" />

*Figure 5: Negative gate trigger for TRIAC with lamp load*

*Testing TRIAC Firing Angle*

config_pulse(&ccr3Data, i, 40);
if(resetState == false)
    i++;
else
    i--;

if(i >= 150)
    resetState = true;
else if(i <= 0)
    resetState = false;

HAL_Delay(500);


Firing angle tested from 0° → 150° and back using HAL_Delay

Pulse width + firing angle limited < 165° to avoid firing next AC cycle

UART Communication

MCU communicates with PC via UART:

Transmit:

HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout);


Receive (interrupt-based):

void interruptRxTx(UART_HandleTypeDef *huart, UartInfo *uart);


Features:

Set target temperature (set temp)

Start/stop temperature logging

Command validation and echo feedback


Figure 6: UART communication example

Temperature Measurement

SPI interface with MAX6675

Resolution: 0.25°C

Functions:

float getTemp(SPI_HandleTypeDef *hspi);
float calculateTemp(uint8_t *TempData);



Figure 7: Testing getTemp function

PID Control

Function:

int findPIDValue(PidInfo *pidInfo, double actualTemp, uint32_t currentTime);


Inputs: actual temperature, target temperature, PID constants (Kp, Ki, Kd)

Output: firing angle for TRIAC

PID Tuning Method:

Start with Kp, Ki, Kd = 0

Increase Kp until oscillation is observed

Adjust Ki to stabilize integral

Adjust Kd to reduce overshoot

Demo Videos:

Kp=10, Ki=0.2, Kd=0: YouTube Demo

Kp=10, Ki=0.2, Kd=0.5: YouTube Demo
