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
https://1drv.ms/i/c/55457142c1bb9d78/EfbkvH8f5zNAqqfYrpcehC8BxR0wnj5WAkPA4ryN4ACmUg

*Figure 3: Input AC vs zero-cross output signal*

---

### TRIAC Triggering via Timer
The MCU uses **Timer 2 output compare** to trigger the TRIAC based on zero-cross detection.  
- Prescaler: 36 
- Timer clock: 2 MHz (0.5 µs per tick)  
- Full sine wave: 40,000 ticks (50 Hz)
 <img width="529" height="473" alt="Screenshot 2025-10-05 203655" src="https://github.com/user-attachments/assets/07810583-d686-4f05-a341-bc487ef99682" />

![Timer Output Compare](https://trello-attachments.s3.amazonaws.com/5cee4d3b0bae3033dfba95f5/5d09d2f828969932f64ef562/c40675b38a1cc59b268e83d3d170c1e2/output_compare.PNG)
*Figure 4: Output compare mode on Timer 2*

#### Key Functions
```c
void update_CCR3(uint32_t CCR1);
void TriacTriggerCallback(CCRxData *ccr3Data);
void config_pulse(CCRxData *ccr3Data, uint32_t firingAngle, uint32_t width);
HAL_TIM_OC_Start_IT(&htim2,TIM_CHANNEL_1);
