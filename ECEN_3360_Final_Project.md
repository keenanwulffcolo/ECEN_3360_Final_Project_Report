# ECEN 3360 Final Project

## Introduction

In this project, we seek to accomplish a very simple task: find the speed of a spinning bicycle tire, and come up with a useful way to display that information on a phone. We decided to use the NXP LPC1115 along with a Bluetooth UART transmitter to send the speed to a smartphone. We originally had planned on using the MPU-6050 I2C gyroscope to acquire angular velocity data, but after a late-stage transition, we decided on using a infrared photodiode combined with an infrared LED to find a period and calculate out the velocity on the board using some simple calculations. Our original timetable is below:

| Date         | Checkpoint                            |
| ------------ | ------------------------------------- |
| Apr 12, 2018 | Accurately reporting gyroscope values |
| Apr 24, 2018 | Accurately reporting linear velocity  |
| May 1, 2018  | Bluetooth communication working       |
| May 8, 2018  | Final Product                         |
| May 9, 2018  | Final Report                          |

With the addition of a few goals (namely, the inclusion of laser cutting acrylic for component housing), and the transition of some others (the aforementioned pivot to infrared technology), this timetable proved more flexible than originally thought

## Background

The Universal asynchronous reciever-transmitter (UART) protocol is one of the bases of communication between components in our project. UART allows two-way serial communication between two devices, and is asynchronous, meaning it does so without a shared clock between the two devices. Instead, the devices operate on predetermined and shared baud rate, which defines the frequency of bits sent across between devices. All characters are sent in a packet, with a configurable size of 5-8 data bits, as well as a start bit, one or two stop bits, and often a parity bit. Since UART lines are by default high voltage, the start bit needs to be low in order to start a transmission. A typical packet looks like the following:

![UART-Packet.png](UART-Packet.png)

In our project, the UART is transmitted through Bluetooth, with the use of of a UART Bluetooth transmitter as an intermediary between the Android phone and the microcontroller. Bluetooth is too complicated of a standard to explain fully, but it is a wireless communication standard made for short range applications. Bluetooth works over a fixed 2.4 GHz frequency, which means there may be many devices in any given room also operating on the same frequency. Core to the Bluetooth specification, then, is a process called pairing, which allows devices to choose each other

The process that leads to pairing starts with one or both devices advertising, or putting a packet into the air containing information such as the device name and manufacturer, and flags giving the support and intended use of the device. The master device search through the advertising packets, and selects the correct one, often with user inputs. It is at this point that pairing is initiated.

Once the two devices are paired, the master device chooses a profile in which to operate. Profiles are protocol for general use-cases associated with Bluetooth. There are protocols for headsets, medical devices, video distribution, and more, but we're concerned in this project with the Serial Port Profile (SPP), which allows Bluetooth to emulate a traditional RS-232 serial port, and by extension, UART.

On the other end of our project, there is a ultraviolet photodiode and ultraviolet (UV) light emitting diode (LED) pair. The LED emits UV light when put in forward bias, and the photodiode acts as a "receiver;" it overcomes the reverse bias put on it when receiving UV light, and thus starts conducting current.

Finally, there was one technology cut from our project: the I2C gyroscope. I2C is a synchronous master-slave interface where each slave is assigned a unique 7-bit identifier. I2C operates over 8-bit packets, where the first packet in each transmission consists of the device identifier and a single bit indication of whether the master intends to read or write. In the case of the MPU-6050, that is followed by an address indicating which register to read or write. If it is a read, the slave communicates back the read data.

## Design

The data flow of our original design has information coming from the gyroscope and communicating with the microcontroller, where it underwent simple calibration. Then, the calibrated data went out via UART to the Bluetooth transmitter, which relayed the same data over Bluetooth, utilizing the Serial Port Profile. There, the data is displayed on the phone.

This changed slightly with the introduction of the UV photodiode system. Here, we mounted the UV LED to the fork on the bike frame in line to where the photodiode rested on the microcontroller, which is mounted on the wheel. The LED was connected to a power source, also mounted to the frame, in series with an 820 $$\Omega$$ resistor, meaning current of 1.8 $$mA$$:



Our photodiode is in reverse bias with a 3.3 $$V$$ pin on the microcontroller and is followed by a 1 $$M\Omega$$ pull down resistor. To avoid interference from the idle voltage on the input pin, we put a comparator circuit on the output, set to flip at 1.75 voltgis:



This drives a GPIO interrupt on the microcontroller, which records the time since the last GPIO interrupt, utilizing the time count from a 32-bit onboard timer set to interrupt every millisecond. The relevant code is below. 


To house the circuitry we designed, we laser cut  two sheets of acrylic using the laser cutter in the ITLL. We cut out a square for each of the microcontroller, Bluetooth module, MPU-6050, and battery bank seperately on one sheet. We held everything in place using zip ties. on the other sheet, we cut holes for zip ties to attach to the spokes of a bike wheel, and attached the two sheets with velcro. This made it so that we could slide the somponent sheet in and out through the spokes, while everything stayed in place when the bike was moving. A picture can be seen below. 

## Results

In our project, we did accomplish the goal we set out to. We made a bike speedometer that accurately read us the speed the bike was traveling at. Our overall functionality was correct, but we did not complete everything we had hoped. 

We ran into problems with the MPU-6050 module once we used the USB power bank to to power everything. We had the desired functionality when plugged into the debugger, but not without. We were still able to power on the chip, and configure the registers we needed. We also read these configuration registers to check the right values. However, when we read the gyroscope data registers they returned zero values. We weren’t certain of the cause of this, and tried everything we could to diagnose this. Because of these issues, we decided to use an infrared LED and an infrared photodiode sensor to measure the speed of the bike.  

The infrared LED and photdiode pair ex=nded up being fairly accurate. We used an interrupt triggered timer to measure the time between rotations, and had to simply multiply that by a conversion factor. The only problem with this was practicality. When you are outside, there is a lot of infrared light from the sun. When the sun is shining on the photodiod, we get very unpredictable results as the interrupt is constantly being triggered. To alleviate this, we could have made a sun shield for the photodiode, but did not have the time. 

If we were to attempt this project again, we don't think our approach would change drastically. We would want to have more knowledge of the MPU-6050, and figure out why that wasn't working. Other than that, we feel our approach would've worked very well had we gotten it working. We were very pleased with out final product for the time we put in. We estimate that we put in about 20 hours of work into the project. A good portion of that was trying to debug the problems with the MPU-6050, and near the end when we decided to change our aproach to the infrared LED and photodiode pair. 

### Cost Analysis

| Component | Cost |
| -------- | ---- |
| AGS 2600mAh USB Power bank | $7.99 |
| GY-521 MPU-6050 | $4.66 |
| HC-06 Bluetooth Serial Module | $8.99 |
| NXP LPC1115 | $26.57 |
| IR LED | $1.00 |
| IR Sensor | $2.00 |
| Total | 51.21 |

## Conclusion


## Code
Main.c
```C
#define CONFIG_ENABLE_DRIVER_ADC 1
#include "driver_config.h"
#include "target_config.h"
#define CONFIG_ENABLE_DRIVER_TIMER32 1
#include "timer32.h"
#include "uart.h"
#include "GPIO.h"
#include "adc.h"
#include "i2c.h"
#define BF_SYSTICK_COUNTFLAG    (1<<16)
#include "mpu6050.h"
extern volatile uint32_t UARTCount;
extern volatile uint8_t UARTBuffer[BUFSIZE];
extern volatile uint32_t NewCount;
extern volatile uint16_t timer_new;
int16_t GyroBuffer[6];
char MainState = 0;

int main (void) {
	  /* Basic chip initialization is taken care of in SystemInit() called
	   * from the startup code. SystemInit() and chip settings are defined
	   * in the CMSIS system_<part family>.c file.
	   */

  /* NVIC is installed inside UARTInit file. */
  UARTInit(9600);

  UARTCount = 0;
  Timer_config();
  GPIO_config();
  Timer_start(0);
  GPIO_LED_setHigh(3);//set AD0 Low
#if 0  //code for I2C communication
  if ( I2CInit( (uint32_t)I2CMASTER ) == FALSE )	/* initialize I2c */
  {
	while (1);				/* Fatal error */
  }

  while(~MainState){	//wait for start signal from phone
  MainState = getChar();
  }
#endif

  /*UNUSED MPU-6050 config code
  mpu6050Power();//initialize Gyro Module
  mpu6050CommTest();
  mpu6050Write(0x23, 0x71);//enable 3-axis Gyro FIFO and ACC FIFO
  mpu6050Write(0x1B, 0x18);//set Full scale range to +/- 250/s(0x08=500/s, 0x10=1000/s, 0x18=2000/s)
  mpu6050Write(0x1C, 0x18);*/

  while (1) 
  {				/* Loop forever */
	  /*UNUSED I2C communication code
	   * mpu6050ReadGyro(0x43,GyroBuffer);//Read Gyro
	   * UARTsendString(GyroBuffer, 6);
	   */
	  //printf("%d, %d, %d \n",GyroBuffer[0], GyroBuffer[1], GyroBuffer[2]);
	  
	  uint32_t total;
	  uint16_t newval;
	  GPIO_LED_setHigh(7);
	  total = (uint32_t)((uint32_t)165*(uint32_t)6283*(uint32_t)1000/(uint32_t)timer_new);
	  newval = (uint16_t)(total/1000);
	  char *nString = itoa(newval);
	  sendString(nString);
	  sendString("\n");


	  GPIO_LED_setLow(7);
	  int i;
	  for(i = 1; i<1000000; i++);
  }
}
```
Timer.c
```C
#include "GPIO.h"
volatile uint16_t us_count;
void sendString(char * inString){
	  LPC_UART->IER = IER_THRE | IER_RLS;			/* Disable RBR */
	uint32_t length = 0;
	while(inString[length] != '\0'){
		length++;
	}
	UARTSend((uint8_t *)inString, length);
	LPC_UART->IER = IER_THRE | IER_RLS | IER_RBR;	/* Re-enable RBR */
}

void Timer_config(void){
	LPC_SYSCON->SYSAHBCLKCTRL |= (1<<9);
	LPC_TMR32B0->TCR = 0x2;
	LPC_TMR32B0->PR = 0x0; // set pr
	LPC_TMR32B0->MCR =  0x03; //trigger interrupt if TC = MR0 and clear MR0
	LPC_TMR32B0->MR0 = 48000; // trigger every 1 ms
	NVIC_EnableIRQ(TIMER_32_0_IRQn);
	LPC_SYSCON->SYSAHBCLKCTRL |= (1<<10);
}

void Timer_start(uint8_t timerNum){
	switch(timerNum){
		case 0:
			LPC_TMR32B0->TCR = 0x1;
			break;
		case 1:
			LPC_TMR32B1->TCR = 0x1;
			break;
	}
}

void Timer_stop(uint8_t timerNum){
	switch(timerNum){
		case 0:
			LPC_TMR32B0->TCR &= ~0x1;
			break;
		case 1:
			LPC_TMR32B1->TCR &= ~0x1;
			break;
	}
}


void TIMER32_0_IRQHandler(void){
	LPC_TMR32B0->TCR &= ~0x1;
	LPC_TMR32B0->IR = (1<<0); // reset interrupt
	us_count++;
	LPC_TMR32B0->TCR |= 0x1;
}

char outString[4];
char teststr[] = "0123456789";
char* itoa(uint16_t num){
	uint16_t i;
	uint16_t num2;
	for(i = 0; i < 5; i++){
		num2 = num%10;
		outString[4-i] = teststr[num2];
		num = num/10;
	}
	return outString;
}
```
GPIO.c
```C
#include "GPIO.h"
extern volatile uint16_t us_count;
volatile uint16_t timer_new;
uint8_t GPIO_config(void){
	/* Enable AHB clock to the GPIO domain. */
	LPC_SYSCON->SYSAHBCLKCTRL |= (1<<6);
	// LED Port 0 Pin 7(red), 9(blue), 8(green)
	LPC_GPIO0->DIR |= ((0x1)<<3 |
					   (0x1)<<7 |
					   (0x1)<<9 |
					   (0x1)<<8 );
	//
	LPC_GPIO2->DIR = 0;
	LPC_GPIO2->IS &= ~(1 << 6);
	LPC_GPIO2->IBE &= ~(1 << 6);
	LPC_GPIO2->IEV &= ~(1 << 6);
	LPC_GPIO2->IE |= (1 << 6);
	NVIC_EnableIRQ(EINT2_IRQn);
	if(((LPC_GPIO0->RIS&0x02)>>1) == 0){
		//return 1;
	}
	return 0;
}
void GPIO_LED_setHigh(uint8_t pin){
	LPC_GPIO0->DATA &= ~(1<<pin);
}
void GPIO_LED_setLow(uint8_t pin){
	LPC_GPIO0->DATA |= (1<<pin);
}
void GPIO_startLED(){
	Timer_start(0);
}
void GPIO_stopLED(){
	Timer_stop(0);
	GPIO_LED_setLow(7);
}
void PIOINT2_IRQHandler(void){
	if(LPC_GPIO2->MIS & (1<<6)){
		LPC_GPIO2->IC |= (1<<6);//clear interrupt
		if(us_count > 10){
			timer_new = us_count;
		}
		us_count = 0;
		LPC_GPIO0->DATA ^= (1<<8);
	}
}
```
mpu6050.c
```C
#include "mpu6050.h"
#include "i2c.h"
#include "driver_config.h"
#include "target_config.h"
#include "GPIO.h"
#include "type.h"
extern volatile uint32_t I2CCount;
extern volatile uint8_t I2CMasterBuffer[BUFSIZE];
extern volatile uint8_t I2CSlaveBuffer[BUFSIZE];
extern volatile uint32_t I2CMasterState;
extern volatile uint32_t RdIndex;
extern volatile uint32_t I2CReadLength, I2CWriteLength;
/* MPU6050 read data, can be used to grab data from sensor*/
void mpu6050ReadGyro(uint8_t startRegAddr, int16_t *GyroBuffer){
	RdIndex = 0;
    // Clear buffers
    uint32_t i, ERR_CODE;
    for (i = 0; i < BUFSIZE; i++) {
        I2CMasterBuffer[i] = 0x00;
        I2CSlaveBuffer[i] = 0x00;
    }
    //uint8_t tempBuffer[6];
    // Write to MPU6050 sensor: start to read from which sensor
    // TO-DO
    I2CWriteLength = 2;
    I2CReadLength = 1;
    //X AXIS
	I2CMasterBuffer[0] = MPU6050_ADDR;
	I2CMasterBuffer[1] = startRegAddr;		/* address */
	I2CMasterBuffer[2] = MPU6050_ADDR | RD_BIT;
	ERR_CODE = I2CEngine();
	GyroBuffer[0] = I2CSlaveBuffer[0]<<8;

	I2CMasterBuffer[1] = startRegAddr +1;		/* address */
	ERR_CODE = I2CEngine();
	GyroBuffer[0] |= I2CSlaveBuffer[0];
}

void mpu6050ReadTest(uint8_t regAdd) {
        // Clear buffers
        uint32_t i, ERR_CODE;
        for (i = 0; i < BUFSIZE; i++) {
            I2CMasterBuffer[i] = 0x00;
            I2CSlaveBuffer[i] = 0x00;
        }
        /* Read who am i register for testing */
        // Tell MPU6050 sensor: what is your name? -- Read the WHO_AM_I register
        // TO-DO
      I2CWriteLength = 2;
	  I2CReadLength = 1;
	  I2CMasterBuffer[0] = MPU6050_ADDR;
	  I2CMasterBuffer[1] = regAdd;		/* address */
	  I2CMasterBuffer[2] = MPU6050_ADDR | RD_BIT;
	  ERR_CODE = I2CEngine();
	  if(ERR_CODE == 11){
		  //printf("Start Condition Never Generated\n");
	  }
	  UARTSend(I2CSlaveBuffer, 1);//send who_am_i to phone
}
/* MPU6050 write data, can be used to configure register*/
void mpu6050Write(uint8_t regAdd,uint8_t regValue) {
        // Clear buffers
        uint32_t i, ERR_CODE;
        for (i = 0; i < BUFSIZE; i++) {
            I2CMasterBuffer[i] = 0x00;
            I2CSlaveBuffer[i] = 0x00;
        }
        // Write to MPU6050: set a value to a register
        // TO-DO
        I2CWriteLength = 3;
        I2CReadLength = 0;
        I2CMasterBuffer[0] = MPU6050_ADDR;
        I2CMasterBuffer[1] = regAdd;		/* address */
        I2CMasterBuffer[2] = regValue;
        ERR_CODE = I2CEngine();
        if(ERR_CODE==11){
        	GPIO_LED_setHigh(0x8);//set Blue LED
        }
    }

void mpu6050Power(void) {
        // Clear buffers
        uint32_t i, ERR_CODE;

        for (i = 0; i < BUFSIZE; i++) {
            I2CMasterBuffer[i] = 0x00;
            I2CSlaveBuffer[i] = 0x00;
        }
        // Write to MPU6050: set a value to a register
        // TO-DO
      I2CWriteLength = 3;
	  I2CReadLength = 0;
	  I2CMasterBuffer[0] = MPU6050_ADDR;
	  I2CMasterBuffer[1] = PWR_MGMT_1;		/* address */
	  I2CMasterBuffer[2] = PWR_WAKE_DATA;
	  ERR_CODE = I2CEngine();
	  //printf("Error Code %x\n", ERR_CODE);
	  if(ERR_CODE==11){
	          	GPIO_LED_setHigh(0x9);//set Blue LED
	  }
    }
/* MPU6050 read who_am_i register, can be used to test I2C communication*/
void mpu6050CommTest(void) {
        // Clear buffers
        uint32_t i, ERR_CODE;
        for (i = 0; i < BUFSIZE; i++) {
            I2CMasterBuffer[i] = 0x00;
            I2CSlaveBuffer[i] = 0x00;
        }
        /* Read who am i register for testing */
        // Tell MPU6050 sensor: what is your name? -- Read the WHO_AM_I register
        // TO-DO
      I2CWriteLength = 2;
	  I2CReadLength = 1;
	  I2CMasterBuffer[0] = MPU6050_ADDR;
	  I2CMasterBuffer[1] = WHO_AM_I;		/* address */
	  I2CMasterBuffer[2] = MPU6050_ADDR | RD_BIT;
	  ERR_CODE = I2CEngine();
	  if(ERR_CODE == 11){
		  //printf("Start Condition Never Generated\n");
	  }
        // Print the value stored in WHO_AM_I register
        //printf("Who_am_I = 0x%X ?\n",I2CSlaveBuffer[0]);
	  //UARTSend(0xAA, 1);
	  UARTSend(I2CSlaveBuffer, 1);
    }
```
