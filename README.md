Download Link: https://assignmentchef.com/product/solved-ee337-lab7-adc-interfacing-with-pt-51-using-spi-2
<br>
<ol>

 <li>Learn embedded C</li>

 <li>Understanding the Serial Peripheral Interface (SPI) protocol</li>

 <li>Write driver code for interfacing ADC IC TLV1543 with Pt-51 using SPI</li>

 <li>Integrating ADC and LCD driver programs to display an analog value using Pt-51</li>

</ol>

<h1>Introduction</h1>

SPI (Serial Peripheral Interface) is a synchronous, serial communication protocol used for communicating between microcontrollers and peripherals (or between 2 or more microcontrollers). In SPI communication, there is a master device (typically the microcontroller), and a slave device (typically a peripheral or other microcontroller). It uses shift registers for converting parallel data to serial and four lines for communication. They are two data lines viz MOSI (Master Out Slave In), MISO (Master In Slave Out), a CLK, and a Select line to choose the device you wish to communicate to.

In this experiment, we will use TLV1543, a 11 channel, 10-bit switched-capacitor, successiveapproximation ADC. The analog signal to be applied may be single ended or differential. Any of the 11 analog input channels can be selected by sending address of corresponding channel serially to the address pin of ADC. The ADC produces 10 bit digital output which is available serially on the Dout pin of the ADC. Please refer to the datasheet of TLV1543 for more details.

Please note that this experiment on-wards we shall use embedded C for programming.

<h1>Homework</h1>

(Read the following documents in this order)

<ol>

 <li>Read the supporting document ‘SPI intro.pdf’</li>

 <li>Read Serial Peripheral Interface (page 89) part of AT89C5131 datasheet, understandthe timing diagram, SPCON register, SPSTA register</li>

 <li>Read TLV1543 datasheet, give special attention to timing diagrams, modes of operationof TLV, allowable max frequency</li>

 <li>Read this handout.</li>

 <li>Read the document on “Embedded C”.</li>

 <li>Go through all shared codes.</li>

</ol>

<h1>Circuit diagram</h1>

Figure 1: Circuit Diagram

<table width="517">

 <tbody>

  <tr>

   <td width="47"> </td>

   <td colspan="2" width="216">Pt-51 Board</td>

   <td width="43"> </td>

   <td colspan="2" width="212">TLV 1543 ADC</td>

  </tr>

  <tr>

   <td width="47"><strong>Pin</strong></td>

   <td width="84"><strong>Function</strong></td>

   <td width="132"><strong>Description</strong></td>

   <td width="43"><strong>Pin</strong></td>

   <td width="84"><strong>Function</strong></td>

   <td width="127"><strong>Description</strong></td>

  </tr>

  <tr>

   <td width="47">P1.7</td>

   <td width="84">MOSI</td>

   <td width="132">Serial Output</td>

   <td width="43">17</td>

   <td width="84">Address</td>

   <td width="127">Serial Input</td>

  </tr>

  <tr>

   <td width="47">P1.6</td>

   <td width="84">SCK</td>

   <td width="132">Serial Clock O/p</td>

   <td width="43">18</td>

   <td width="84">I/O clk</td>

   <td width="127">Serial Clock I/P</td>

  </tr>

  <tr>

   <td width="47">P1.5</td>

   <td width="84">MISO</td>

   <td width="132">Serial Input</td>

   <td width="43">16</td>

   <td width="84">Data out</td>

   <td width="127">Serial O/p</td>

  </tr>

  <tr>

   <td width="47">P1.4</td>

   <td width="84"><em>SS</em></td>

   <td width="132">Slave Select</td>

   <td width="43">15</td>

   <td width="84"><em>CS</em></td>

   <td width="127">Chip Select</td>

  </tr>

 </tbody>

</table>

Table 1: SPI pin connections between Pt-51 and TLV1543

The pins on the Pt-51 board marked as SPI are used to connect peripherals with SPI connectivity, i.e., the ADC TLV 1543 in this experiment. Understand the circuit in Fig. 1 and connections in Table 1 for the required hardware connection.

<h1>Software architecture</h1>

The program flow and function calls can be understood from Fig. 2.

Figure 2: Software architecture for interfacing the SPI peripherals. You can ignore orange coloured blocks (filter.h and TLV5616.h) for now as we will use them in the next experiment.

In this experiment, we will be performing several tasks and hence we shall follow a modular coding practice (writing header files for the tasks and functions (subroutines) for each subtask). This practice makes it easier to read, understand and debug the code. It also makes it possible to scale up the project by including additional header files. (In fact, we will be adding another module to this experiment during the next lab session.)

<h1>Function descriptions</h1>

This section describes all the functions required to be written for the interfacing of ADC to read analog DC input voltage connected to one of its channels and display it on the LCD. Note that each header file and functions in them perform specific tasks. The functions are written such that they take input data from other function and return processed data.

<ol>

 <li><strong>c :</strong></li>

</ol>

This file includes all necessary header files which have function definitions that will be used. The microcontroller executes <em>main() </em>function from this file. This function calls all necessary functions sequentially. Use <em>main.c </em>as it is, there is no need to edit this file. Flow of operations in <em>main() </em>is as follows:

<ul>

 <li>It calls initialization functions for all peripherals which are <em>spi init() </em>(for SPI module of Microcontroller AT89C5131), <em>adc init() </em>(for ADC IC TLV1543) and <em>lcd init() </em>(for LCD).</li>

 <li>Display “Channel-00” on first line using functions available in <strong>h</strong>.</li>

 <li>Steps (e) to (g) are repeated infinitely using while loop so that recent input samplewill be displayed on LCD.</li>

 <li>Call <em>adc() </em>function for communicating with the ADC, which will return digital values in the range 0 to 1023. This value is proportional to the input voltage. Note that input voltage to be applied is 0 to 3.3 V.</li>

 <li>Appropriately scale the digital value (0 to 1023) to lie in the 0 to 3300 mV range.</li>

 <li>Call <em>int to string() </em>function with appropriate argument to convert sampled input value to an ascii string that can be displayed on the LCD.</li>

 <li>Call function <em>lcd write string() </em>to display the ascii value on the 2nd line of LCD.</li>

</ul>

<ol start="2">

 <li><strong>h :</strong></li>

</ol>

Functions in this file control all operations related to AT89C5131 SPI module. Task of this module is to facilitate SPI communication between master (Microcontroller) and slave (SPI device – ADC). Data from all interfaced SPI devices are gathered by calling function from this file and passed to other functions for further processing. So if we have to switch to another microcontroller plateform, only <strong>spi.h </strong>(driver) file needs to be changed and processing functions can remain the same. The file <strong>spi.h </strong>contains following functions.

<ul>

 <li><em>spi init():</em>

  <ol>

   <li>Parameters to be passed: None</li>

   <li>Parameters to be returned: None</li>

  </ol></li>

</ul>

<ul>

 <li>Task:</li>

</ul>

Function initialises SPI module in AT89C5131 by configuring SPCON, IEN0 and IEN1 register.

<ul>

 <li><em>spi trx 16 bit()</em>:</li>

</ul>

<ol>

 <li>Parameters to be passed: unsigned int – 16 bit data to be sent from master to slave. ii. Parameters to be returned: unsigned int – 16 bit data from slave.</li>

</ol>

iii. Task:

In AT89C5131, spdat (Transmit and receive data buffer) is an 8 bit SFR which allows 8 bit data to be transmitted or received. To transmit or receive 16 bit data, two 8 bit data frames may be used one after the other. 16 bit data passed to this function from other functions will be unpacked (split) into two bytes. These two bytes are transmitted to slave, simultaneously master will receive two bytes from the slave. Note that most significant byte is transmitted first and then the least significant byte. Similarly, the master will receive the most significant byte first and then the least significant byte. These two received bytes are then packed (joined) into 16 bit data. This data word will be returned by this function.

(c) <em>spi interrupt()</em>:

<ol>

 <li>Parameters to be passed: None</li>

 <li>Parameters to be returned: None</li>

</ol>

<ul>

 <li>Task: This is the ISR (Interrupt Service Routine) for SPI interrupt. Thisfunction will be called after completion of transfer of 8 bits of data to the slave. The 8 bits received from the slave are stored in a temporary global variable which will be used by <em>spi trx 16 bit() </em></li>

</ul>

<ol start="3">

 <li><strong>h:</strong></li>

</ol>

Functions in this file are specific to TLV1543. These functions control and enable data transfer between microcontroller and ADC slave. Handling of control signals and data processing specific to this IC is facilitated by this header file. Functions from <strong>spi.h </strong>are called by functions in this file. Hence <strong>spi.h </strong>is included in <em>tlv1543.h</em>. Configuration parameters specific to TLV1543 are set in function in this file. Following are functions in this file.

<ul>

 <li><em>adc init()</em>:

  <ol>

   <li>Parameters to be passed: None</li>

   <li>Parameters to be returned: None</li>

  </ol></li>

</ul>

<ul>

 <li>Task:</li>

</ul>

Control signals related to TLV1543 are initialized here.

<ul>

 <li><em>adc()</em>:

  <ol>

   <li>Parameters to be passed: None</li>

   <li>Parameters to be returned: unsigned int – two bytes of data correspondingto the analog input received. iii. Task:</li>

  </ol></li>

</ul>

This function triggers communication between microcontroller and ADC slave.

Following tasks are executed sequentially.

<ul>

 <li>Multiple slaves (if connected) might need different settings of configuration of same SPI module of master. These slave specific requirements are taken care of by functions in slave specific driver (Here <em>adc()</em>).</li>

 <li>Enable ADC slave.</li>

 <li>A two byte data frame having address of input channel A0 is formed (Check timing diagram of SPI communication for this). This data is then passed to function <em>spi trx 16 bit()</em>.</li>

 <li><em>spi trx 16 bit() </em>will take care of SPI communication between microcontroller and ADC slave and return sampled ADC value from that channel. 5) Disable ADC slave.</li>

</ul>

6) Data word returned by function <em>spi trx 16 bit() </em>is processed to make it in correct format (Check timing diagram of SPI communication for this).

This data word is then returned by this function.

<ol start="4">

 <li><strong>h:</strong></li>

</ol>

You have already used LCD in previous experiments using assembly code. In this experiment, we shall write C functions for doing the same. <strong>lcd.h </strong>header includes all functions related to LCD module. Following are some key functions in <strong>lcd.h </strong>(other functions are self explanatory).

<ul>

 <li><em>lcd init()</em>:

  <ol>

   <li>Parameters to be passed: None</li>

   <li>Parameters to be returned: None</li>

  </ol></li>

</ul>

<ul>

 <li>Task:</li>

</ul>

This function initializes LCD. It clears LCD and move cursor to start of 1st line of LCD.

<ul>

 <li><em>lcd cmd()</em>:

  <ol>

   <li>Parameters to be passed: unsigned int – 1 byte command for LCD.</li>

   <li>Parameters to be returned: None</li>

  </ol></li>

</ul>

<ul>

 <li>Task:</li>

</ul>

This function activates control signal specific to sending command to LCD. It transfers 1 byte command to LCD module. Clear screen, move courser to specific location etc., are few commands. See datasheet of LCD for more commands.

<ul>

 <li><em>lcd write char()</em>:

  <ol>

   <li>Parameters to be passed: unsigned char – 1 byte, ASCII value of the character.</li>

   <li>Parameters to be returned: None</li>

  </ol></li>

</ul>

<ul>

 <li>Task:</li>

</ul>

Activate control signal specific to printing a character on LCD. This function transmits ASCII value of the character to LCD. LCD prints it at current position of cursor. Cursor will be moved to next position.

<ul>

 <li><em>int to string( , )</em>:

  <ol>

   <li>Parameters to be passed: unsigned int – 16 bits of positive integer data and</li>

  </ol></li>

</ul>

unsigned char * – pointer to an ASCII string location.

<ol>

 <li>Parameters to be returned: None</li>

</ol>

<ul>

 <li>Task:</li>

</ul>

This function converts positive integer value to decimal number. This decimal number is then converted to string of ASCII. This string is stored at location pointed by the pointer.

(e) <em>lcd write string()</em>:

<ol>

 <li>Parameters to be passed: unsigned char * – Pointer to an ASCII string which</li>

</ol>

is to be displayed on the LCD. ii. Parameters to be returned: None

iii. Task:

This function uses function <em>lcd write char() </em>to send all characters from ASCII string to LCD.

<h1>Labwork</h1>

<ol>

 <li>Complete function <em>spi init() </em>to configure SPCON, IENO and IEN1 registers of AT89C5131.

  <ul>

   <li>microcontroller SPI module as Master</li>

   <li>Free the SS pin for a general-purpose (microcontroller does not requires chip selectas we are configuring it in master mode)</li>

   <li>Select the Master clock rate as Fclk/16 (by choosing appropriate values of SPR0,SPR1, SPR2 we can get different baud rates)</li>

   <li>Select appropriate setting for serial clock polarity and clock phase: Refer timingdiagrams (Figures 3 and 4 in this handout) of SPI communication of microcontroller and TLV1543 (Refer datasheets of both ICs for more information).</li>

   <li>Enable SPI module of the microcontroller.</li>

   <li>Enable SPI interrupt (<strong>IEN1</strong>-Interrupt enable register) (Read the datasheet of AT89C5131a for more information)</li>

   <li>Set enable all interrupt (EA) bit (<strong>IEN0</strong>– Interrupt enable register) (Refer datasheet of AT89C5131a for more information)</li>

  </ul></li>

 <li>Connect circuit as mentioned in Fig. 1. Note that input is given to A0 channel. Displaythe measured voltage (0 to 3.3 V) on the LCD in the format: On 1st line ”Channel-00:” and on 2nd line ”XXXXX mV” (XXXXX is sampled input voltage from channel A0 in mV).</li>

 <li>Modify function <em>adc() </em>such that it will take 1 byte data as an input argument. This argument takes value from 0 to 13 which corresponds to input channel from A0 to A10 and 3 internal test inputs of ADC (Refer TLV1543 datasheet page no 5 and 6). Function <em>adc() </em>should return sampled input corresponds to ADC channel number which was passed to it. Update <em>main() </em>function such that input from any two channels (A0 – A10) of ADC and three internal test inputs will be displayed periodically with an interval of 2 secs on LCD in following format: On 1st line ”Channel-XX:” (XX channel number: 0 to 13) and on 2nd line ”XXXX mV” (XXXXX is sampled input voltage in mV from ADC channel number XX).</li>

</ol>

<h1>Formula for voltage conversion</h1>

Approximate procedure for analog to digital conversion of input voltage is given as (Refer figure number-7 for ideal conversion characteristics)

Where,

V<em><sub>IN </sub></em>= Analoginput voltage

V<em><sub>REF </sub></em>= Analog reference voltage

<h1>Details from datasheets of AT89c5131 and TLV1543</h1>

Figure 3: SPI timing diagram for AT89C5131

Figure 4: SPI timing diagram for TLV1543 IC

Figure 5: SPCON Register: AT89C5131

Figure 6: Note regarding SPI SFR, AT89C5131

Figure 7: TLV1543 Analog to digital conversion