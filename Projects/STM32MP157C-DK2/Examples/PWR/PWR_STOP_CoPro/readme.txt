/**
  @page PWR_STOP_CoPro PWR Example

  @verbatim
  ******************************************************************************
  * @file    Examples/PWR/PWR_STOP_CoPro/readme.txt
  * @author  MCD Application Team
  * @brief   Description of the PWR_STOP_CoPro example.
  ******************************************************************************
  *
  * Copyright (c) 2021 STMicroelectronics.
  * All rights reserved.
  *
  * This software is licensed under terms that can be found in the LICENSE file
  * in the root directory of this software component.
  * If no LICENSE file comes with this software, it is provided AS-IS.
  *
  *
  ******************************************************************************
  @endverbatim

@par Example Description

How to enter the CSTOP and STOP modes using CM4 core
(aka Co-processor) and wake up from this mode by using external wakeup interrupt.

This project deals with CM4 firmware and requires Linux
OS running on CA7 core (aka Master processor).

Before running this firmware:
  - CA7 through Linux sets System and CM4 clocks.
  - CA7 loads and powers on CM4.
In this firmware :
  - CM4 sets EXTI14 line to the the User push-button on PA.14
    and it is configured to generate an interrupt upon key press.
  - The SysTick is programmed to generate an interrupt each 1 ms and in the
    SysTick interrupt handler, LED7 is toggled in order to indicate whether the
    CM4 is in CSTOP mode or RUN mode.
  - 5 seconds after start-up, the CM4 automatically enters CSTOP
    mode and LED7 stops toggling.
    Note: Just before going into CSTOP CM4 prepares CoPro side:
          it backs up CM4 clock configuration, disables IO compensation cell,
          masks ITs (except RCC wake up IT) and clears Low Power flags.
  - The User push-button can be pressed at any time to wake-up the CM4.
  - The software then comes back in RUN mode for 5 seconds before automatically
    entering CSTOP mode again.
    Note: Just after waking up CM4 restores its context depending of
          operating mode tested (explained further below).

  - These steps are repeated in an infinite loop.

Notes:
LED7 is used to monitor the CM4 state as follows:
 - LED7 ON: configuration failed or error occured (CM4 will go to an infinite loop) 
 - LED7 toggling: CM4 in RUN mode
 - LED7 off : CM4 in CSTOP mode
 - Firmware requires Linux console to start & run example. Following
   command should be done in Linux console to run the example :
    > ./fw_cortex_m4.sh start

--------------------------------------------------------------------------------
- The following operating modes can be tested using this example -
--------------------------------------------------------------------------------

1)  System RUN Mode with CA7 in CRun and CM4 in CSTOP
    ----------------------------------------------------------------------------
    Objective: Testing CM4 CSTOP low power mode and CM4
               wakes up with EXTI14 line
    1. CA7 loads CM4 firmware
    2. CM4 firmware is executed as explained before. User wakes up CM4
       using EXTI14 line connected to User push-button
        Note: On CM4 side, right after waking up, RCC wake up IRQ is treated
          (HW has completely recovered from Low Power)
          As platform wasn't on STOP mode, CM4 doesn't restores its
          clock configuration (it's not needed).
          CM4 re-enables IO compensation cell and unmasks all ITs
          User push-button EXTI14 IRQ is then treated

2) System STOP Mode with CA7 in CSTOP and CM4 in CSTOP - CM4 wakes up first
   -----------------------------------------------------------------------------
    Objective: Testing System STOP Mode with CM4 in charge of re-configuring
               its clock configuration
               CM4 wakes up before CA7

    1. CA7 loads CM4 firmware
    2. User puts CA7 into CSTOP mode (using console)
        > systemctl suspend
    3. CM4 firmware is executed as explained before. User wakes up CM4
       using EXTI14 line connected to User push-button
       Note: As both cores were on CSTOP mode, system was on STOP mode.
             On CM4 side, right after waking up, RCC wake up IRQ is treated
             (HW has completely recovered from Low Power)
             As platform was on STOP mode, CM4 restores its clock
             configuration.
             CM4 re-enables IO compensation cell and unmasks all ITs
             User push-button EXTI14 IRQ is then treated
    4. User wakes up CA7 using WAKE UP button
       Note: After waking up, CA7 sets CM4 clock configuration
             too but as it's the same there's no any conflict.

3) System STOP Mode with CA7 in CSTOP and CM4 in CSTOP - CA7 wakes up first
   -----------------------------------------------------------------------------
    Objective: Testing System STOP Mode with CA7 configuring clocks first
               CA7 wakes up before CM4

    1. CA7 loads CM4 firmware
    2. User puts CA7 into CSTOP mode (using console)
        > systemctl suspend
    3. CM4 firmware is executed as explained before but user wakes up
       CA7 first using WAKE UP button and then user wakes up CM4
       using EXTI14 line connected to User push-button
       Note: As both cores were on CSTOP mode, system was on STOP mode.
             After waking up, CA7 sets up CM4 clock configuration.
             On CM4 side, right after waking up, RCC wake up IRQ is treated
             (HW has completely recovered from Low Power)
             As platform was on STOP mode, CM4 sets up its clock
             configuration too but as it's the same there's no any conflict.
             CM4 re-enables IO compensation cell and unmasks all ITs
             User push-button EXTI14 IRQ is then treated


@note This example can not be used in DEBUG mode due to the fact
      that the Cortex-M4 core is no longer clocked during low power mode
      so debugging features are disabled.

@note Care must be taken when using HAL_Delay(), this function provides accurate
      delay (in milliseconds) based on variable incremented in SysTick ISR.
      This implies that if HAL_Delay() is called from a peripheral ISR process,
      then the SysTick interrupt must have higher priority (numerically lower)
      than the peripheral interrupt. Otherwise the caller ISR process will be
      blocked. To change the SysTick interrupt priority you have to use
      HAL_NVIC_SetPriority() function.

@note The application needs to ensure that the SysTick time base is always set
      to 1 millisecond to have correct HAL operation.

@note By default the I/O compensation cell is not used. However when the
      I/O output buffer speed is configured in 50 MHz mode and above, it
      is recommended to use the compensation cell for a slew rate control
      on I/O tf(IO)out/tr(IO)out commutation to reduce the I/O noise on
      the power supply.

@note Only a few number of peripherals that could be used by CM4 will use I/O
      configured at 50 MHz and above (Eg.: SDMMC) but this example shows how to
      use IO compensation mechanism as enabling/disabling this is needed on STOP mode.

@par Keywords

Power, PWR, CSTOP mode, STOP mode, wake-up, Interrupt, low power
mode.

@par Directory contents

  - Examples/PWR/PWR_STOP_CoPro/Inc/lock_resource.h           Header file for lock_resource.c
  - Examples/PWR/PWR_STOP_CoPro/Inc/main.h                    Header file for main.c
  - Examples/PWR/PWR_STOP_CoPro/Inc/stm32mp1xx_hal_conf.h     HAL Configuration file
  - Examples/PWR/PWR_STOP_CoPro/Inc/stm32mp1xx_it.h           Header for stm32mp1xx_it.c
  - Examples/PWR/PWR_STOP_CoPro/Inc/stm32mp1xx_stm32mp15xx_disco_conf.h    BSP configuration file
  - Examples/PWR/PWR_STOP_CoPro/Src/system_stm32mp1xx.c       STM32MP1xx system clock configuration file
  - Examples/PWR/PWR_STOP_CoPro/Src/stm32mp1xx_it.c           Interrupt handlers
  - Examples/PWR/PWR_STOP_CoPro/Src/main.c                    Main program
  - Examples/PWR/PWR_STOP_CoPro/Src/stm32mp1xx_hal_msp.c      HAL MSP module
  - Examples/PWR/PWR_STOP_CoPro/Src/lock_resource.c           Remoteproc lock resource file

@par Hardware and Software environment

  - This example runs on STM32MP1xx devices

  - This example has been tested with STMicroelectronics STM32MP157C-DK2
    board and can be easily tailored to any other supported device
    and development board.

  - STM32MP157C-DK2 set-up:
    - LED7 connected to PH.7 pin
    - User push-button connected to pin PA.14 (EXTI14)
    - CA7 WAKE UP button connected to PA.0 pin

@par How to use it ?

In order to make the program work, you must do the following :
 - Open your preferred toolchain
 - Rebuild all files and load your image into target memory
 - Run the example

 * <h3><center>&copy; COPYRIGHT STMicroelectronics</center></h3>
 */
