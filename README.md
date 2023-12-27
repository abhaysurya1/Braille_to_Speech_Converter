# Braille_to_Speech_Converter
______________UART

#include<LPC17xx.h>
void delay(unsigned int r1);
void UART0_Init(void);

unsigned int  i ;
unsigned char *ptr, arr[] = "Hello world\r";
#define THR_EMPTY 0x20											//U0THR is Empty 
int main(void)
{
	UART0_Init();
	while(1)
	{
		ptr = arr;
		while ( *ptr != '\0')
		{ 
			while ((LPC_UART0->LSR & THR_EMPTY) != THR_EMPTY) ; //Is U0THR is EMPTY??
			LPC_UART0->THR = *ptr++;			
		}
		for(i=0;i<=60000;i++);
	} 
}

void UART0_Init(void)
{
	LPC_SC->PCONP |= 0x00000008;			//UART0 peripheral enable
	LPC_PINCON->PINSEL0 = 0x00000050;
	LPC_UART0->LCR = 0x00000083;			//enable divisor latch, parity disable, 1 stop bit, 8bit word length
	LPC_UART0->DLM = 0X00; 
	LPC_UART0->DLL = 0x1A;      			//select baud rate 9600 bps for 4Mhz
	LPC_UART0->LCR = 0X00000003;			//Disable divisor latch
	LPC_UART0->FCR = 0x07;						//FIFO enable,RX FIFO reset,TX FIFO reset
}



____________________________________________________________________________________________________________________

DCM DIRECTION


#include <LPC17xx.H>
void Clock_Wise(void);
void AClock_Wise(void);

unsigned long i;
int main(void)
{
	LPC_PINCON->PINSEL1 = 0x00000000;	//P0.26 GPIO, P0.26 controls dir
	LPC_PINCON->PINSEL3 = 0x00000000;	//P1.24 GPIO
	LPC_GPIO0->FIODIR |= 0x04000000;	//P0.26 output
	LPC_GPIO1->FIODIR |= 0x01000000;	//P1.24 output

	while(1)
	{
		Clock_Wise();
		for(i=0;i<300000;i++);
		AClock_Wise();
		for(i=0;i<300000;i++);
	}		//end while(1)
}			//end main

void Clock_Wise(void)
{
	LPC_GPIO1->FIOCLR = 0x01000000;		//P1.24 Kept low to off DCM
	for(i=0;i<10000;i++);							//delay to componsate inertia
	LPC_GPIO0->FIOSET = 0x04000000;		//coil is on
	LPC_GPIO1->FIOSET = 0x01000000;		//motor in on
}										//end void Clock_Wise(void)

void AClock_Wise(void)
{
	LPC_GPIO1->FIOCLR = 0x01000000;		//P1.24 Kept low to off DCM
	for(i=0;i<10000;i++);				//delay to componsate inertia
	LPC_GPIO0->FIOCLR = 0x04000000;		//coil is off
	LPC_GPIO1->FIOSET = 0x01000000;		//Motor is on
}										//end void AClock_Wise(void)


_______________________________________________________________________________________


DCM SPEED


#include <lpc17xx.h>
void pwm_init(void);
void PWM1_IRQHandler(void);
unsigned long int i;
unsigned char flag,flag1;

int main(void)
{   
	pwm_init();
	while(1);
}//end of main

void pwm_init(void)
{
	LPC_SC->PCONP = (1<<6);						//PWM1 is powered
	LPC_PINCON->PINSEL3 = 0x00020000;	//pwm1.5 is selected for the pin P1.24  
	LPC_PWM1->PR  = 0x00000000;      	//Count frequency : Fpclk 
	LPC_PWM1->PCR = 0x0002000;      	//select PWM5 single edge and PWM5 output is enabled 
	LPC_PWM1->MCR = 0x00000003;      	//Reset and interrupt on PWMMR0
	LPC_PWM1->MR0 = 60000;           	//setup match register 0 count 
	LPC_PWM1->MR5 = 0x00000100;      	//setup match register MR5 
	LPC_PWM1->LER = 0x000000FF;      	//enable shadow copy register
	LPC_PWM1->TCR = 0x00000002;      	//RESET COUNTER AND PRESCALER
	LPC_PWM1->TCR = 0x00000009;      	//enable PWM and counter
	NVIC_EnableIRQ(PWM1_IRQn);
}
void PWM1_IRQHandler(void)
{
	LPC_PWM1->IR = 0xff; 				//clear the interrupts
	if(flag == 0x00)
    {
			LPC_PWM1->MR5 += 100;
			LPC_PWM1->LER = 0x000000FF;
			if(LPC_PWM1->MR5 >= 57000)						//for 10% of duty cycle MR5=3000,for 50% MR5=15000 ,for 90% MR5= 27000
				{
							flag1 = 0xff;
							flag = 0xff;
							LPC_PWM1->LER = 0x000000fF;
				}	
		 for(i=0;i<8000;i++);
	}
    else if(flag1 == 0xff)
    {
			LPC_PWM1->MR5 -= 100;
			LPC_PWM1->LER = 0x000000fF;
			if(LPC_PWM1->MR5 <= 0x300)					//1% of duty cycle MR5=300
			{
				flag  = 0x00;
				flag1 = 0x00;
				LPC_PWM1->LER = 0X000000fF;
			}
		 for(i=0;i<8000;i++);
	 }
}

_____________________________________________________________________________________________________



STEPPER MOTOR


#include <LPC17xx.H>
void clock_wise(void);
void anti_clock_wise(void);
unsigned long int var1,var2;
unsigned int i=0,j=0,k=0;

int main(void)
{
	LPC_PINCON->PINSEL4 = 0x00000000;		//P2.0 to P2.3 GPIO
	LPC_GPIO2->FIODIR = 0x0000000F;			//P2.0 to P2.3 output
	while(1)
	{
		for(j=0;j<50;j++)           			//50 times in Clock wise Rotation
			clock_wise();				
		
		for(k=0;k<65000;k++);        			//Delay to show  anti_clock Rotation 
		
		for(j=0;j<50;j++)          				//50 times in  Anti Clock wise Rotation
			anti_clock_wise();

		for(k=0;k<65000;k++);        			//Delay to show clock Rotation 
	} 																	//End of while(1)
} 																		//End of main
void clock_wise(void)
{
		var1 = 0x00000001;         				//For Clockwise
    for(i=0;i<=3;i++)         				//for A B C D Stepping
	{
	   LPC_GPIO2->FIOCLR =  0X0000000F;
	   LPC_GPIO2->FIOSET =  var1;
		 var1 = var1<<1;        					//For Clockwise
     for(k=0;k<15000;k++); 						//for step speed variation         
  }
}
void anti_clock_wise(void)
{ 
		var1 = 0x0000008;      						//For Anticlockwise
    for(i=0;i<=3;i++)       					//for A B C D Stepping
    {
	    LPC_GPIO2->FIOCLR =  0X0000000F;
			LPC_GPIO2->FIOSET =  var1;
			var1 = var1>>1;      						//For Anticlockwise
      for(k=0;k<15000;k++); 					//for step speed variation 
    }
}
______________________________________________________________________________________________


ADC 


#include<LPC17xx.h>
#include "lcd.h"
#include<stdio.h>
#define	Ref_Vtg		3.300
#define	Full_Scale	0xFFF				//12 bit ADC

int main(void)
{
	unsigned int adc_temp;
	unsigned int i;
	float in_vtg;
	unsigned char vtg[7],dval[7], blank[]="   ";
	unsigned char Msg3[11] = {"ANALOG IP:"};
	unsigned char Msg4[12] = {"ADC OUTPUT:"};

   lcd_init();
   LPC_PINCON->PINSEL3 |= 0xC0000000;																	//P1.31 as AD0.5
	 LPC_SC->PCONP |= (1<<12);																					//enable the peripheral ADC
	
	temp1 = 0x80;
	lcd_com();
	delay_lcd(800);
	lcd_puts(&Msg3[0]);

	temp1 = 0xC0;
	lcd_com();
	delay_lcd(800);
	lcd_puts(&Msg4[0]);

	while(1)
	{
		LPC_ADC->ADCR = (1<<5)|(1<<21)|(1<<24);															//0x01200001;//ADC0.5, start conversion and operational	
		for(i=0;i<2000;i++);																								//delay for conversion
		while((adc_temp = LPC_ADC->ADGDR) == 0x80000000);										//wait till 'done' bit is 1, indicates conversion complete
		adc_temp = LPC_ADC->ADGDR;
		adc_temp >>= 4;
		adc_temp &= 0x00000FFF;																							//12 bit ADC
		in_vtg = (((float)adc_temp * (float)Ref_Vtg))/((float)Full_Scale);	//calculating input analog voltage
		sprintf(vtg,"%3.2fV",in_vtg);																				//convert the readings into string to display on LCD
		sprintf(dval,"%x",adc_temp);
		for(i=0;i<2000;i++);

		temp1 = 0x8A;
		lcd_com();
		delay_lcd(800);
		lcd_puts(&vtg[0]);

		temp1 = 0xCB;
		lcd_com();
		lcd_puts(&blank[0]);

		temp1 = 0xCB;
		lcd_com();
		delay_lcd(800);
		lcd_puts(&dval[0]);

		for(i=0;i<200000;i++);
		for(i=0;i<7;i++)
		vtg[i] = dval[i] = 0x00;
		adc_temp = 0;
		in_vtg = 0;
	}
}

 
__________________________________________________________________________________________________________


DAC SINE WAVE


#include <LPC17xx.H>

int count=0,sinevalue,value;
unsigned char sine_tab[49]=
	{ 0x80,0x90,0xA1,0xB1,0xC0,0xCD,0xDA,0xE5,0xEE,0xF6,0xFB,0xFE,
      0xFF,0xFE,0xFB,0xF6,0xEE,0xE5,0xDA,0xCD,0xC0,0xB1,0xA1,0x90,
      0x80,0x70,0x5F,0x4F,0x40,0x33,0x26,0x1B,0x12,0x0A,0x05,0x02,
      0x00,0x02,0x05,0x0A,0x12,0x1B,0x26,0x33,0x40,0x4F,0x5F,0x70,0x80};

int main(void)
{
	LPC_PINCON->PINSEL0 &= 0xFF0000FF ;						// Configure P0.0 to P0.15 as GPIO
  LPC_GPIO0->FIODIR  |= 0x00000FF0 ;
	LPC_GPIO0->FIOMASK = 0XFFFFF00F;
 
	count = 0;
	while(1)
	{
		for(count=0;count<48;count++)
		{
			sinevalue = sine_tab[count];//+0X10 ;
			value= 0x00000FF0 & (sinevalue << 4);
			LPC_GPIO0->FIOPIN = value;
		}
	}
}
______________________________________________________________________________________________________


DAC SQUARE WAVE 

#include <LPC17xx.H>

void delay(void);

int main ()
{
		LPC_PINCON->PINSEL0 &= 0xFF0000FF ;						// Configure P0.4 to P0.11 as GPIO
    LPC_GPIO0->FIODIR  |= 0x00000FF0 ;
		LPC_GPIO0->FIOMASK = 0XFFFFF00F;
	while(1)
    {
    	LPC_GPIO0->FIOPIN  = 0x00000FF0 ;
        delay();
        LPC_GPIO0->FIOCLR  = 0x00000FF0 ;
        delay();
    }
}   
 
void delay(void)
{
	unsigned int i=0;
   	for(i=0;i<=9500;i++);
}
_________________________________________________________________________________________

DAC TRIANGLE

#include <LPC17xx.H>

int main ()
{
	unsigned long int temp=0x00000000; 
	unsigned int i=0;
  
  LPC_PINCON->PINSEL0 &= 0xFF0000FF ;						// Configure P0.4 to P0.11 as GPIO
	LPC_GPIO0->FIODIR  |= 0x00000FF0 ;
	LPC_GPIO0->FIOMASK = 0XFFFFF00F;
	   
    while(1)
    {
    	//output 0 to FE 
        for(i=0;i!=0xFF;i++)
        {
        	temp=i;
        	temp = temp << 4;
        	LPC_GPIO0->FIOPIN = temp;
        }
       	// output FF to 1   
        for(i=0xFF; i!=0;i--)
        {
        	temp=i;
        	temp = temp << 4;
        	LPC_GPIO0->FIOPIN = temp;
        }
	}//End of while(1)
}//End of main()

_____________________________________________________________________________________________________________


KEYPAD LCD

#include <LPC17xx.h>
#include "lcd.h"
void scan(void);

unsigned char Msg1[20] = "PG VLSI";
unsigned char Msg2[13] = "KEY PRESSED=";
unsigned char col,row,var,flag,key,*ptr;
unsigned long int i,var1,temp,temp3;

unsigned char SCAN_CODE[16] = {0x1E,0x1D,0x1B,0x17,
 							  	0x2E,0x2D,0x2B,0x27,
							  	0x4E,0x4D,0x4B,0x47,
							  	0x8E,0x8D,0x8B,0x87};

unsigned char ASCII_CODE[16] = {'0','1','2','3',
 								 '4','5','6','7',
								 '8','9','A','B',
								 'C','D','E','F'};

int main(void)
{
	LPC_PINCON->PINSEL3 = 0x00000000; 			//P1.20 to P1.23 MADE GPIO
	LPC_PINCON->PINSEL0 = 0x00000000;  			//P0.15 as GPIO
	LPC_PINCON->PINSEL1 = 0x00000000; 			//P0.16 t0 P0.18 made GPIO
	LPC_GPIO0->FIODIR &= ~0x00078000; 			//made INput P0.15 to P0.18 (cols)
	LPC_GPIO1->FIODIR |= 0x00F00000; 				//made output P1.20 to P1.23 (rows)
	LPC_GPIO1->FIOSET = 0x00F00000;  

	lcd_init();

	temp1 = 0x80;														//point to first line of LCD
	lcd_com();
	delay_lcd(800);
	lcd_puts(&Msg1[0]);	 										//display the messsage

	temp1 = 0xC0;														//point to first line of LCD
	lcd_com();
	delay_lcd(800);
	lcd_puts(&Msg2[0]);	 										//display the messsage

	while(1)
	{
		while(1)
		{
			for(row=1;row<5;row++)
			{
				if(row == 1)
				var1 = 0x00100000;
				else if(row == 2)
				var1 = 0x00200000;
				else if(row == 3)
				var1 = 0x00400000;
				else if(row == 4)
				var1 = 0x00800000;
			
				temp = var1;

				LPC_GPIO1->FIOSET = 0x00F00000;
				LPC_GPIO1->FIOCLR = var1;

				flag = 0;
				scan();
				if(flag == 1)
				break;
		
			} 																	//end for(row=1;row<5;row++)

			if(flag == 1)
			break;
		
		}							 												//2nd while(1)

		for(i=0;i<16;i++)
		{
			if(key == SCAN_CODE[i])
			{
				key = ASCII_CODE[i];
				break;
			} 																//end if(key == SCAN_CODE[i])

		}																		//end for(i=0;i<16;i++)

		temp1 = 0xCC;
		lcd_com();
		delay_lcd(800);
		lcd_puts(&key);
		
	}																		//end while 1
}																			//end main

void scan(void)
{
 	unsigned long temp3;
	temp3 = LPC_GPIO0->FIOPIN;	
	temp3 &= 0x00078000;
	if(temp3 != 0x00078000)
	{
		for(i=0;i<500;i++);
		temp3 = LPC_GPIO0->FIOPIN;	
		temp3 &= 0x00078000;
		if(temp3 != 0x00078000)
		{
			flag = 1;
			temp3 >>= 15;									//Shifted to come at LN of byte
			temp >>= 16;									//shifted to come at HN of byte
			key = temp3|temp;	
		}																//2nd if(temp3 != 0x00000000)
	}																	//1st if(temp3 != 0x00000000)
}																		//end scan

________________________________________________________________________________________________________________________

PWM

#include <LPC17xx.H>
void pwm_init(void);
void PWM1_IRQHandler(void);
unsigned long int i;
unsigned char flag,flag1;
int main(void)
{     
	pwm_init();
	while(1);
}																		//end of main

void pwm_init(void)
{
	LPC_SC->PCONP = (1<<6);						//PWM1 is powered
	LPC_PINCON->PINSEL7 = 0x000C0000;	//pwm1.2 is selected for the pin P3.25
	  
	LPC_PWM1->PR  = 0x00000000;      	//Count frequency : Fpclk 
	LPC_PWM1->PCR = 0x00000400;      	//select PWM2 single edge 
	LPC_PWM1->MCR = 0x00000003;      	//Reset and interrupt on PWMMR0
	LPC_PWM1->MR0 = 30000;           	//setup match register 0 count 
	LPC_PWM1->MR2 = 0x00000100;      	//setup match register MR1 
	LPC_PWM1->LER = 0x000000FF;      	//enable shadow copy register
	LPC_PWM1->TCR = 0x00000002;      	//RESET COUNTER AND PRESCALER
	LPC_PWM1->TCR = 0x00000009;      	//enable PWM and counter

	NVIC_EnableIRQ(PWM1_IRQn);
}

void PWM1_IRQHandler(void)
{
	LPC_PWM1->IR = 0xff; 							//clear the interrupts

	if(flag == 0x00)
    {
		LPC_PWM1->MR2 += 100;
		LPC_PWM1->LER = 0x000000FF;

		if(LPC_PWM1->MR2 >= 27000)			//Is Duty Cycle 90% ??
		{
        	flag1 = 0xff;
        	flag = 0xff;
        	LPC_PWM1->LER = 0x000000fF;
		}
	}
    else if(flag1 == 0xff)
    {
		LPC_PWM1->MR2 -= 100;
		LPC_PWM1->LER = 0x000000fF;
    
 		if(LPC_PWM1->MR2 <= 0x300)			//Is Duty Cycle 1% ??
		{
			flag  = 0x00;
			flag1 = 0x00;
			LPC_PWM1->LER = 0X000000fF;
		}
	}
}

__________________________________________________________________________________________________________


EXT INTERRUPT

#include<LPC17xx.h>
void EINT3_IRQHandler(void);
unsigned char int3_flag=0;
int main(void)
{
	LPC_PINCON->PINSEL4 |= 0x04000000;		//P2.13 as EINT3
	LPC_PINCON->PINSEL4 &= 0xFCFFFFFF;		//P2.12 GPIO for LED
	LPC_GPIO2->FIODIR = 0x00001000;				//P2.12 is assigned output
	LPC_GPIO2->FIOSET = 0x00001000;				//Initiall LED is kept on
	
	LPC_SC->EXTINT = 0x00000008;					//writing 1 cleares the interrupt, get set if there is interrupt
	LPC_SC->EXTMODE = 0x00000008;					//EINT3 is initiated as edge senitive, 0 for level sensitive
	LPC_SC->EXTPOLAR = 0x00000000;				//EINT3 is falling edge sensitive, 1 for rising edge
																				//above registers, bit0-EINT0, bit1-EINT1, bit2-EINT2,bit3-EINT3	
	NVIC_EnableIRQ(EINT3_IRQn);						//core_cm3.h
	  
	while(1) ;
}

void EINT3_IRQHandler(void)
{
		LPC_SC->EXTINT = 0x00000008;				//cleares the interrupt
	
		if(int3_flag == 0x00)								//when flag is '0' off the LED
		{
			LPC_GPIO2->FIOCLR = 0x00001000;
			int3_flag = 0xff;
		}										
		else																//when flag is FF on the LED
		{
			LPC_GPIO2->FIOSET = 0x00001000;
			int3_flag = 0;
		}
}

________________________________________________________________________________________________________________________

7 Segment

#include <LPC17xx.h>
unsigned int delay, count=0, Switchcount=0,j;

unsigned int Disp[16]={0x000003f0, 0x00000060, 0x000005b0, 0x000004f0, 0x00000660,0x000006d0,
					   0x000007d0, 0x00000070, 0x000007f0, 0x000006f0, 0x00000770,0x000007c0,
					   0x00000390, 0x000005e0, 0x00000790, 0x00000710 };

#define ALLDISP  0x00180000												//Select all display
#define DATAPORT 0x00000ff0												//P0.4 to P0.11 : Data lines connected to drive Seven Segments
int main (void)
{
	LPC_PINCON->PINSEL0 = 0x00000000;   
	LPC_PINCON->PINSEL1 = 0x00000000;
	LPC_GPIO0->FIODIR =	0x00180ff0;
	
	while(1)
	{
		LPC_GPIO0->FIOSET |= ALLDISP;
		LPC_GPIO0->FIOCLR =	0x00000ff0;	 							// clear the data lines to 7-segment displays
		LPC_GPIO0->FIOSET = Disp[Switchcount];   			// get the 7-segment display value from the array
				
			 for(j=0;j<3;j++)
			for(delay=0;delay<30000;delay++);	 					// delay   
			                                   
				Switchcount++;
				if(Switchcount == 0x10)										// 0 to F has been displayed ? go back to 0
			    {
				   Switchcount = 0;
				   LPC_GPIO0->FIOCLR  =	0x00180ff0;				
			    }
	}
}

_____________________________________________________________________________________________________________________________

RELAY AND BUZZER


#include <LPC17xx.H>
int main(void)
{
	LPC_PINCON->PINSEL1 = 0x00000000;								//P0.24,P0.25 GPIO
	LPC_GPIO0->FIODIR = 0x03000000;									//P0.24 configured output for buzzer,P0.25 configured output for Relay/Led
   while(1)
   {
				if(!(LPC_GPIO2->FIOPIN & 0x00000800))			//Is GP_SW(SW4) is pressed??
				{
					 LPC_GPIO0->FIOSET = 0x03000000;				//relay on
				}	
				else
				{
					 LPC_GPIO0->FIOCLR = 0x03000000;				//relay off
				}
	 }
}																									//end int main(void)

