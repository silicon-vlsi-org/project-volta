# Project Volta : SPI, I2C, SRAM and Bandgap Reference in 0.6um CMOS Technology.
In this project 12 undergraduate students designed, simulated and tested a Serial Peripheral Interface (SPI) for a 32 byte Static Random Access Memory (SRAM) using an 0.6um CMOS Technology which can be used as a low power consuming device by integrating it with Microcontrollers (MCUs) and peripheral ICs ,for example IOT sensors. The project was accomplished using Tanner EDA tools to design, simulate and layout the circuits. This project also contained the other IPs 

# Resources
- [SPI Accessible SRAM](https://www.dropbox.com/s/vuq0kentfr0qxja/2019-0730-SPI-SRAM-report-grp1-subhra-smita.pdf) : A project writeup by Subhra Mahapatra and Smita Panda.
- [SRAM Controller Design](https://www.dropbox.com/s/lr6c93vjr3xvl0v/2019-0619-ProjectDescrip-SPI-controller.pdf): A brief outline of the controller design which generates the SRAM signals from the SPI signals.

# Static Random Access Memory (SRAM)
## Abstract
This paper presents the detailed design and implementation of a Static Random-Access Memory (SRAM) in CMOS technology. The entire operation of the SRAM (read, write, and control) is done through Serial Peripheral Interface (SPI), an industry-standard serial protocol, without the requirement of any built-in clock or bias, making it compact and low-power. This SRAM is specifically suitable for Internet-of-Things (IoT) applications with slow access rates and low power consumption like IoT-based environment sensors. For the purpose of demonstration, a 32-byte SRAM was designed and fabricated in 0.6um CMOS technology and successfully tested for its full functionality after fabrication.

Keywords- Static Random Access Memory, Serial Peripheral Interface, Internet-of-Things, CMOS, Sensor node.
## Introduction
A typical Internet-of-Things (IoT) sensor node contains four main devices. As shown in Fig. 1, these four devices are; a sensor, a microcontroller unit (MCU), a wireless device (Bluetooth, WiFi, LoRa), and static random access memory (SRAM). Since sensor nodes work on low data rates and many sensors could interface to the controller, communicate among themselves over a serial bus such as the Serial Peripheral Interface (SPI) or Inter IC Communication (I2C) protocol to save space and keep the cost down.
<p align="center">
  <img src="/images/Fig1-Block-diagram-of-a-typical-IoT-sensor-node.png">
  Fig.1: Block diagram of a typical IoT sensor node
</p>


The MCU serves as the controller for the IoT node to negotiate all transactions between devices. The wireless device is responsible for sending sensor data to the nearest internet gateway and, receiving control instructions from the gateway as well. The sensor measures environmental and other parameters (temperature, humidity, position, tilt, color, etc.) and the data is digitized and transmitted over the SPI/ I2C serial communication bus. The SRAM is used to buffer local data when a gateway is not available for communication.

Since these sensor nodes are low-cost and low-power, the peripherals also need to be the same. This paper describes the design and implementation of a low-power SPI-accessible SRAM implemented in a 0.6μm CMOS technology. The innovation of this work lies in creating and controlling all the signals required for the SRAM without the use of internal clocks or biases, deriving all control signals from the SPI signals themselves instead, thereby making it a compact and scalable circuit that can be made to consume very little power with slower access time.

The remainder of this paper is described as follows. Section-II describes the complete SPI-based SRAM architecture and its working principles. Section-III describes the design and implementation of different blocks and the related simulation results. Section-IV presents the test bench setup and measurement results. Section-V concludes the work.

## SPI-Based SRAM Architecture
This section describes the SRAM architecture and the read and write operations.
### A. SRAM Architecture
The architecture of the SRAM with the serial interface is shown in Fig. 2. The SRAM consists of the array of the core 6-transistor (6T) SRAM cells along with the pre-charge circuits, row decoders, column decoders, and the sense amplifiers. The SPI interface converts the incoming serial signals to their appropriate control signals to read and write the data from the SRAM using the serial bus. The SRAM core consists of 32 bytes of 8-bit words laid out as a 16x16 array of standard 6T CMOS memory cells [1][2]. Each cell is individually accessed by selecting the corresponding row and column using their respective decoders. The word lines are selected using the 4-bit LSB of the 5-bit SRAM address A[3:0] and two 8-bit columns are selected using the MSB of the address A[4]. After selecting the individual cells, the corresponding bit lines are used to read from the cell or write to it. During the read operation, the bit lines are first pre-charged to a pre-determined value of the supply voltage minus the threshold voltage of the NMOS device (VDD-VTN), then the pre-charge circuit is disabled, and the SRAM cell is selected after that. The difference in the voltage between the complementary bit lines represents the data in the SRAM cell and typically this differential voltage is a fraction of the supply voltage, not enough to drive a digital gate directly. A differential sense amplifier is used to amplify this small differential voltage to the supply rails VDD or ground, and then the output of the sense amplifier can drive digital gates directly. The output is fed back to the SPI controller which then converts the parallel 8-bit data to a serial output complying with the SPI protocol. During the write operation, the SPI controller converts the serial input data to 8-bit parallel data, and the corresponding SRAM cells are selected, as in the read operation, but the column decoders now connect the bit lines to the 8-bit drivers to write the data instead of sensing it. Once the data is written, all the cells are de-selected, and the data remain stored until the power is applied to the IC or overwritten by another write operation.
<p align="center">
  <img src="/images/Fig2-Architecture-of-SPI-interface-based-SRAM.jpg">
  Fig.2: Architecture of SPI interface based SRAM
</p>

The next two sections detail the architecture of the timing to keep everything synchronous with the SPI clock (SLCK). The SPI protocol has 4 modes of operation (0-3) [3][4] and this SPI controller is designed for Mode-2 where the idle SCLK and CS are high and, the microcontroller sends data at the negative edge of the clock and samples at the positive edge of the clock. To keep the design simple and avoid Static-Time Analysis (STA) for setup/hold time issues, the controller is designed such that it sends and samples data on opposite edges of SCLK. 
### B. Write Operation
Fig. 3 shows the timing diagram of the write operation. After the microcontroller selects the SRAM chip by pulling Chip-Select (CS) low, 2 bytes are transferred serially out on the output port (SDO) with data changing synchronously on every negative edge of the clock (SCLK). The first bit denotes whether the operation reads (1) or writes (0). The next 5 bits are assigned to the address of the SRAM and the last two bits are unused. The second byte contains the 8-bit data that needs to be written to the corresponding address. The controller latches the read/write bit after the first clock cycle and latches the address to the decoders at the end of the 6th clock cycle. Then the controller generates a pre-charge signal (pc) at the positive edge of the 14th cycle for one clock period. The data is latched for write at the positive edge of the last clock cycle (16th) and write control signals (wl, wr, col) are also asserted at the same edge and pulled low when the chip is de-selected (CS is high).
<p align="center">
  <img src="/images/Fig3-Timing-diagram-of-write-operation.png">
  Fig. 3: Timing diagram of write operation
</p>

### C. Read Operation
Fig. 4 shows the timing diagram for the read operation. The first byte of the read operation is the same as the write operation except, the pre-charge signal (pc) is asserted at the positive edge of the 7th clock cycle and the read enable signal (ren) is asserted for one clock cycle at the positive edge of the 8th clock cycle during which the data from the SRAM (DSNS[7:0]) is latched by a shift register at the negative edge of the 9th clock cycle. Once the data is latched, it is shifted bit by bit into the input port of the microcontroller (SDI).
<p align="center">
  <img src="/images/Fig4-Timing-diagram-of-read-operation.png">
  Fig. 4: Timing diagram of read operation
</p>

## SRAM Design and Simulation Results
As discussed in the previous section, the complete SRAM design contains a 6-transistor (6T) SRAM cell, a pre-charge circuit, a row decoder, control logic, a column decoder, and a sense amplifier. This section will discuss the design of the blocks and their simulation results.
### A. 6-Transistor (6T) CMOS SRAM Cell
The core of the SRAM is a memory cell that stores one bit of information. Each cell’s area and power are critical since it decides the area and power of the entire chip. This design uses a standard 6T SRAM cell [1][2]. In principle, it is a back-to-back inverter (M<sub>1</sub>,M_2,M_5,M_6) to store data indefinitely if power is provided to the cell. The access transistors (M_3,M_4) are used to read and write data into the cell. The sizing of the devices is decided by three main factors: the area of the cell, stored data in the memory cells is not corrupted while reading it, and able to overwrite the stored data during write operation. Analysis with the appropriate large-signal equations (saturation/linear) [1], it can be shown that M_3 needs to be stronger than M_5 and, M_1 needs to be stronger than M_3. Since the structure is symmetric, the same constraints apply for M_6,M_4,M_2. Since these constraints are unbounded, additional constraints are needed to choose the size of transistors in the 6T cell which is typically a trade-off between area and speed. In this design, the 6T transistors were sized for minimum area.
<p align="center">
  <img src="/images/Fig5-6T-SRAM-cell-circuit-diagram.png">
  Fig. 5: 6T SRAM cell circuit diagram
</p>

### B. Pre-charge Circuit
Since the output bitlines (bl and blb) of each 6T cell are shared by all the cells in the rows in that column, the parasitic capacitance on those nodes is very large making it impractical for the 6T cells to drive the bitlines to full CMOS voltage levels. Instead, both the bitlines are pre-charged to the same voltage, and a differential amplifier is used to sense the difference between the bitlines to read it. The nodes are also pre-charged before a write operation to reset a previous operation. As shown in Fig. 6, NMOS M_7 and M_8 are used to pre-charge the bitlines to VDD-VTN. Since the sensing mechanism is a differential operation, it is critical for both the bitlines to be equal in voltage for which PMOS M_9 is used. It should be noted that the bitlines are pre-charged to VDD-VTN instead of VDD. It is done to keep the differential sense amplifier active during the pre-charge phase. 
<p align="center">
  <img src="/images/Fig6-Pre-charge-circuit.png">
  Fig. 6: Pre-charge circuit
</p>
    
### C. Row Decoder
Fig. 7 shows a pass-transistor logic based 4:16 row decoder to select any row from the sixteen rows in the SRAM array based on the input address bit configurations. The 4-bit address signals a[3:0] are used to activate the transistors in such a way that, any one of the outputs will be high. For example, if all the address bits are low (0000), then wl[0] output will be high and this will select the 0th row in the SRAM array. Similarly, if all the address bits are high (1111), wl[15] output line will be high and that will select the 15th row of the SRAM array. Since the pass-transistor logic only passes high (VDD), a pull-down is required at the output.
<p align="center">
  <img src="/images/Fig7-Row-decoder.png">
  Fig. 7: Row decoder
</p>

Finally, a buffer is used to convert the output to CMOS levels and drive the large capacitance of the word line. The pull-down was realized using a 1uA current source (I_0). Please note that this was an experimental design to demonstrate a compact design but can be realized using standard static CMOS logic.
### D. Column Decoder Logic
Fig. 8 shows the block diagram of column decoder logic. Based on the address, the column decoder selects the appropriate columns in the SRAM array. In this work, an 8-bit byte of data is either read from or written to the SRAM array. Therefore, from the 16 columns, a set of 8 is chosen based on the MSB of the address a[4]. Depending on if it’s a write (rwn=high) or read (rwn=low) operation, the decoder logic either allows the data from the write driver to be written to the cell or tri-states them. The bitlines are always connected to the input of the sense amplifier so in case of a read operation, the sense amplifiers are simply enabled at the appropriate time
<p align="center">
  <img src="/images/Fig8-Contol-logic-and-column-decoder.jpg">
  Fig. 8: Contol logic and column decoder
</p>

### E. Sense Amplifier
The sense amplifier is used to sense the voltage difference between the bitlines and amplify it to drive the digital circuits. There are different types of sense amplifiers are present [5,6] and used in the SRAM design depending upon the application which is typically a trade-off between area and power. In this work, an analog differential amplifier-based sense amplifier is implemented. Fig. 9 shows the differential amplifier-based sense amplifier. As shown in the figure, during the read operation a small voltage difference between the bitlines ‘bl’ and ‘blb’ is amplified by the sense amplifier and the buffer converts the output to rail-to-rail CMOS voltage. The gain of the amplifier and the threshold of the buffer is designed in tandem to achieve that function.
<p align="center">
  <img src="/images/Fig9-Circuit-diagram-of-the-sense-amplifier.png">
  Fig. 9: Circuit diagram of the sense amplifier
</p>

### F. SPI Interface and SRAM Controller
The core contribution of this work is to create all the control signals for the SRAM using only the SPI signals (SCLK, CS, SDO, SDI) without the need for any internal clock or bias which adds to the time and cost of the design. Fig. 10 shows the architecture of the SPI interface to the SRAM. 
<p align="center">
  <img src="/images/Fig10-Architecture-of-SPI-SRAM-interface.jpg">
  Fig. 10: Architecture of SPI-SRAM interface
</p>

Two 8-bit shift registers form the serial-parallel interface for read and write operation. The 4-bit synchronous counter and a set of comparators are used to keep track of the SPI clock. The SRAM control logic is a Mealy Finite State Machine (FSM) that uses the comparator output and the SPI signals to create the appropriate control logic (pc, wen, ren) for the SRAM read and write operation as detailed in the timing diagrams in Fig. 3 and Fig. 4.
## Test Setup and Measurement Result
Fig. 11 shows the chip layout and chip micrograph. A picture-schematic of the measurement setup is shown in Fig. 12. Analog discovery 2 (AD2) [7], a USB-based multi- instrument including an oscilloscope and logic analyzer, was used to test the read and write function of the SRAM using SPI protocol. Open-source Python API for AD2 was used to generate the SPI signals and the same AD2 was used as an oscilloscope to monitor the SPI signals.
<p align="center">
  <img src="/images/Fig11-Chip-layout-and-chip-micrograph.jpg">
  Fig. 11: Chip layout and chip micrograph
</p>

Fig. 13 shows an example output for an SPI write and read transaction. Date 0xDC is written into location 0x19 and the same location is read to verify the written data. As seen from the measurement result, the first bit on SDO is 0 denoting a write, the next five bit is the address 0x19 and the second word is 0xDC. The last two byte is a read operation of the same location and seen from the SDI output of the SRAM which is 0xDC.
<p align="center">
  <img src="/images/Fig12-Picture-schematic-of-the-test-and-measurement-setup.png">
  Fig. 12: Picture schematic of the test and measurement-setup
</p>
<p align="center">
  <img src="/images/Fig13-Mesurement-result.png">
  Fig. 13: Measurement result of write operation to address 0x19 location followed by read operation of the same location.
</p>

## Conclusion
A SRAM design in 0.6μm CMOS technology is demonstrated to be a viable candidate for IoT-based sensor nodes. Deriving all the internal signals from the SPI signals allows IoT designers to scale the power directly through the access rates. It also keeps the design simple to keep design costs to a minimum. Additionally, this project can also be used as a tapeout-based course project in the undergraduate curriculum to give the students an end-to-end experience in IC design. 0.6um CMOS process is an old and mature process making it a low-cost technology allowing educational institutions to make fabrication cost affordable.
## Acknowledgement
We acknowledge the support of Deepika Kumari, Gautam Kumar, Pragya Tiwari, Sameer Namdeo, ShivaPrasad Das, Suruchi Kumari, Unnati Kumari Gupta, Anshuman Mishra, Sneha Kumari, Manamohan Nath, Prachi Mrudula and Vishal Sao for their help in simulation and layout.
## References
[1]	Leblebici, Y., Kim, C. W., & Kang, S.-M. “CMOS Digital Integrated Circuits Analysis & Design” 4th-Ed McGraw-Hill Ed, 2014.

[2]	H. E. Weste, D. M. Harris “CMOS VLSI Design: A Circuits and Systems Perspective” Fourth edition, Pearson Education, 2010.

[3]	Leens, F. “An introduction to I2C and SPI protocols”. IEEE Instrumentation Measurement Magazine, 12(1), pp. 8–13., 2009 https://doi.org/10.1109/MIM.2009.4762946

[4]	Introduction to SPI Interface | Analog Devices. (n.d.). Retrieved December 21, 2019, from https://www.analog.com/en/analog-dialogue/articles/introduction-to-spi-interface.html.

[5]	I. S. A. Halim, N. H. Basemu, S. L. M. Hassan and A. A. A. Rahim, "Comparative study on CMOS SRAM sense amplifiers using 90nm technology," 2013 International Conference on Technology, Informatics, Management, Engineering and Environment, Bandung, Indonesia, 2013, pp. 171-175

[6]	B. Mohammad, P. Dadabhoy, K. Lin and P. Bassett, "Comparative study of current mode and voltage mode sense amplifier used for 28nm SRAM," 2012 24th International Conference on Microelectronics (ICM), Algiers, Algeria, 2012, pp. 1-6.

[7]	Analog Discovery 2 from https://digilent.com/reference/test-and-measurement/analog-discovery-2/start 

## Design of a 6T SRAM 

## Serial Protocol Interface (SPI)

## SRAM Controller

# Bandgap Reference

# I2C
