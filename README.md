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
### *A. SRAM Architecture*
The architecture of the SRAM with the serial interface is shown in Fig. 2. The SRAM consists of the array of the core 6-transistor (6T) SRAM cells along with the pre-charge circuits, row decoders, column decoders, and the sense amplifiers. The SPI interface converts the incoming serial signals to their appropriate control signals to read and write the data from the SRAM using the serial bus. The SRAM core consists of 32 bytes of 8-bit words laid out as a 16x16 array of standard 6T CMOS memory cells [1][2]. Each cell is individually accessed by selecting the corresponding row and column using their respective decoders. The word lines are selected using the 4-bit LSB of the 5-bit SRAM address A[3:0] and two 8-bit columns are selected using the MSB of the address A[4]. After selecting the individual cells, the corresponding bit lines are used to read from the cell or write to it. During the read operation, the bit lines are first pre-charged to a pre-determined value of the supply voltage minus the threshold voltage of the NMOS device (VDD-VTN), then the pre-charge circuit is disabled, and the SRAM cell is selected after that. The difference in the voltage between the complementary bit lines represents the data in the SRAM cell and typically this differential voltage is a fraction of the supply voltage, not enough to drive a digital gate directly. A differential sense amplifier is used to amplify this small differential voltage to the supply rails VDD or ground, and then the output of the sense amplifier can drive digital gates directly. The output is fed back to the SPI controller which then converts the parallel 8-bit data to a serial output complying with the SPI protocol. During the write operation, the SPI controller converts the serial input data to 8-bit parallel data, and the corresponding SRAM cells are selected, as in the read operation, but the column decoders now connect the bit lines to the 8-bit drivers to write the data instead of sensing it. Once the data is written, all the cells are de-selected, and the data remain stored until the power is applied to the IC or overwritten by another write operation.
<p align="center">
  <img src="/images/Fig2-Architecture-of-SPI-interface-based-SRAM.jpg">
  Fig.2: Architecture of SPI interface based SRAM
</p>

The next two sections detail the architecture of the timing to keep everything synchronous with the SPI clock (SLCK). The SPI protocol has 4 modes of operation (0-3) [3][4] and this SPI controller is designed for Mode-2 where the idle SCLK and CS are high and, the microcontroller sends data at the negative edge of the clock and samples at the positive edge of the clock. To keep the design simple and avoid Static-Time Analysis (STA) for setup/hold time issues, the controller is designed such that it sends and samples data on opposite edges of SCLK. 
### *B. Write Operation*
Fig. 3 shows the timing diagram of the write operation. After the microcontroller selects the SRAM chip by pulling Chip-Select (CS) low, 2 bytes are transferred serially out on the output port (SDO) with data changing synchronously on every negative edge of the clock (SCLK). The first bit denotes whether the operation reads (1) or writes (0). The next 5 bits are assigned to the address of the SRAM and the last two bits are unused. The second byte contains the 8-bit data that needs to be written to the corresponding address. The controller latches the read/write bit after the first clock cycle and latches the address to the decoders at the end of the 6th clock cycle. Then the controller generates a pre-charge signal (pc) at the positive edge of the 14th cycle for one clock period. The data is latched for write at the positive edge of the last clock cycle (16th) and write control signals (wl, wr, col) are also asserted at the same edge and pulled low when the chip is de-selected (CS is high).
<p align="center">
  <img src="/images/Fig3-Timing-diagram-of-write-operation.png">
  Fig. 3: Timing diagram of write operation
</p>

### *C. Read Operation*
Fig. 4 shows the timing diagram for the read operation. The first byte of the read operation is the same as the write operation except, the pre-charge signal (pc) is asserted at the positive edge of the 7th clock cycle and the read enable signal (ren) is asserted for one clock cycle at the positive edge of the 8th clock cycle during which the data from the SRAM (DSNS[7:0]) is latched by a shift register at the negative edge of the 9th clock cycle. Once the data is latched, it is shifted bit by bit into the input port of the microcontroller (SDI).
<p align="center">
  <img src="/images/Fig4-Timing-diagram-of-read-operation.png">
  Fig. 4: Timing diagram of read operation
</p>

## SRAM Design and Simulation Results
As discussed in the previous section, the complete SRAM design contains a 6-transistor (6T) SRAM cell, a pre-charge circuit, a row decoder, control logic, a column decoder, and a sense amplifier. This section will discuss the design of the blocks and their simulation results.
### *A. 6-Transistor (6T) CMOS SRAM Cell*
The core of the SRAM is a memory cell that stores one bit of information. Each cell’s area and power are critical since it decides the area and power of the entire chip. This design uses a standard 6T SRAM cell [1][2]. In principle, it is a back-to-back inverter (M<sub>1</sub>, M<sub>2</sub>, M<sub>5</sub>, M<sub>6</sub>) to store data indefinitely if power is provided to the cell. The access transistors (M<sub>3</sub>, M<sub>4</sub>) are used to read and write data into the cell. The sizing of the devices is decided by three main factors: the area of the cell, stored data in the memory cells is not corrupted while reading it, and able to overwrite the stored data during write operation. Analysis with the appropriate large-signal equations (saturation/linear) [1], it can be shown that M<sub>3</sub> needs to be stronger than M<sub>5</sub> and, M<sub>1</sub> needs to be stronger than M<sub>3</sub>. Since the structure is symmetric, the same constraints apply for M<sub>6</sub>, M<sub>4</sub>, M<sub>2</sub>. Since these constraints are unbounded, additional constraints are needed to choose the size of transistors in the 6T cell which is typically a trade-off between area and speed. In this design, the 6T transistors were sized for minimum area.
<p align="center">
  <img src="/images/Fig5-6T-SRAM-cell-circuit-diagram.png">
  Fig. 5: 6T SRAM cell circuit diagram
</p>

### *B. Pre-charge Circuit*
Since the output bitlines (bl and blb) of each 6T cell are shared by all the cells in the rows in that column, the parasitic capacitance on those nodes is very large making it impractical for the 6T cells to drive the bitlines to full CMOS voltage levels. Instead, both the bitlines are pre-charged to the same voltage, and a differential amplifier is used to sense the difference between the bitlines to read it. The nodes are also pre-charged before a write operation to reset a previous operation. As shown in Fig. 6, NMOS M<sub>7</sub> and M<sub>8</sub> are used to pre-charge the bitlines to VDD-VTN. Since the sensing mechanism is a differential operation, it is critical for both the bitlines to be equal in voltage for which PMOS M_9 is used. It should be noted that the bitlines are pre-charged to VDD-VTN instead of VDD. It is done to keep the differential sense amplifier active during the pre-charge phase. 
<p align="center">
  <img src="/images/Fig6-Pre-charge-circuit.png">
  
  Fig. 6: Pre-charge circuit
</p>
    
### *C. Row Decoder*
Fig. 7 shows a pass-transistor logic based 4:16 row decoder to select any row from the sixteen rows in the SRAM array based on the input address bit configurations. The 4-bit address signals a[3:0] are used to activate the transistors in such a way that, any one of the outputs will be high. For example, if all the address bits are low (0000), then wl[0] output will be high and this will select the 0th row in the SRAM array. Similarly, if all the address bits are high (1111), wl[15] output line will be high and that will select the 15th row of the SRAM array. Since the pass-transistor logic only passes high (VDD), a pull-down is required at the output.
<p align="center">
  <img src="/images/Fig7-Row-decoder.png">
  Fig. 7: Row decoder
</p>

Finally, a buffer is used to convert the output to CMOS levels and drive the large capacitance of the word line. The pull-down was realized using a 1uA current source (I_0). Please note that this was an experimental design to demonstrate a compact design but can be realized using standard static CMOS logic.
### *D. Column Decoder Logic*
Fig. 8 shows the block diagram of column decoder logic. Based on the address, the column decoder selects the appropriate columns in the SRAM array. In this work, an 8-bit byte of data is either read from or written to the SRAM array. Therefore, from the 16 columns, a set of 8 is chosen based on the MSB of the address a[4]. Depending on if it’s a write (rwn=high) or read (rwn=low) operation, the decoder logic either allows the data from the write driver to be written to the cell or tri-states them. The bitlines are always connected to the input of the sense amplifier so in case of a read operation, the sense amplifiers are simply enabled at the appropriate time
<p align="center">
  <img src="/images/Fig8-Contol-logic-and-column-decoder.jpg">
  Fig. 8: Contol logic and column decoder
</p>

### *E. Sense Amplifier*
The sense amplifier is used to sense the voltage difference between the bitlines and amplify it to drive the digital circuits. There are different types of sense amplifiers are present [5,6] and used in the SRAM design depending upon the application which is typically a trade-off between area and power. In this work, an analog differential amplifier-based sense amplifier is implemented. Fig. 9 shows the differential amplifier-based sense amplifier. As shown in the figure, during the read operation a small voltage difference between the bitlines ‘bl’ and ‘blb’ is amplified by the sense amplifier and the buffer converts the output to rail-to-rail CMOS voltage. The gain of the amplifier and the threshold of the buffer is designed in tandem to achieve that function.
<p align="center">
  <img src="/images/Fig9-Circuit-diagram-of-the-sense-amplifier.png">
  Fig. 9: Circuit diagram of the sense amplifier
</p>

### *F. SPI Interface and SRAM Controller*
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
## *References*
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
## Title: A Power and Area Efficient CMOS Bandgap Reference Circuit with an Integrated Voltage-Reference Branch
## *Abstract*
This work presents a compact and low power bandgap voltage reference design using self-biased current mirror circuit. This design eliminates the standard com-plementary-to-absolute-temperature (CTAT) bipolar device in the voltage-reference branch, reducing the bipolar area by 20%. Instead, the design shares the same bipolar device in the main CTAT branch for generating the reference volt-age. An additional benefit of eliminating the voltage-reference branch is the re-duction of total power consumption by approximately 30%. This novel topology reduces power and area of the core bandgap reference circuit without compromis-ing temperature drift performance. Designed, fabricated and functionally tested in a 0.6um CMOS process. The simulation result shows the temperature coefficient of this design is 6.3 ppm/ºC for a temperature range of -40 ºC to 125 ºC. This bandgap reference design occupies a silicon area of 0.018 mm2 and draws an av-erage quiescent current of 2 µA from a supply voltage of 3.3-5V. The simulated flicker voltage noise is 4.34 µV/sqrt-Hz at 10 Hz.

**Keywords:** bandgap, voltage reference, temperature coefficient, self-biased cur-rent mirror, CMOS.
## 1  Introduction
Voltage reference circuits are an essential block in most applications from a simple integrated circuit (IC) to a large System-on-Chip (SoC) ranging from purely digital circuits to mixed-signal applications such as Analog to Digital converters (ADCs), Digital to Analog converters (DACs), phase locked loops (PLLs), low noise amplifiers (LNAs), digital multimeters, battery chargers, low-power IoT sensor nodes, portable data acquisition systems and so on. Since the first bandgap reference (BGR) circuit introduced by Robert Widlar in 1971 [1], BGR has been widely used since it provides a well-defined voltage reference with a very weak dependence on process, voltage, and temperature. Most analog and mixed-signal circuits also require a current reference that sets the internal bias current for the circuits. The BGR can also provide a reference current directly which has positive temperature co-efficient (PTC). For most bias currents, a PTC reference current is sufficient. For circuits demanding more stable current reference can achieve so with some additional circuits [2].

Fig. 1 shows two type of traditional BGR circuit: (a) one using operational amplifier (Op-Amp) and (b) using a self-biased current mirror circuit [3]. The principle of operations is the same in both cases where the nodes ‘A’ and ‘B’ are forced to be the same by (a) the Op-Amp or (b) the self-biased current mirror. Forcing same node voltages makes the voltage drop across R_1 be exactly difference between the base-to-emitter voltage (V_BE) of the two bi-polar transistor provided that, the size of the transistor Q_1=N.Q_2. The voltage across the resistor R_1 produces a proportional-to-absolute-temperature (PTAT) voltage, which is multiplied with a suitable constant and added to V_BE of Q_3 to generate a stable voltage [4] as follows:

V_REF=V_BE3+V_T.R_2/R_1 .ln⁡(N) -------------------------------(1)

Where, V_REF is the output reference voltage and V_T is the thermal voltage of the semiconductor.

<p align="center">
  <img src="/images/FIG1_BGR.png">
  Fig. 1.  Traditional BGR circuits; (a) using Op-Amp, (b) using self-biased current mirror.
</p>

Typically, Op-Amp based BGR is preferred over self-biased for better power sup-ply rejection (PSR) performance and lower supply requirement. Although the self-biased BGR may have a lower performance in those two metrics, it is a simpler design consuming less area and power while achieving almost similar temperature drift per-formance. In most IC or SoC designs the self-biased BGR performance may suffice to allow less design time, lower risk, lower area and power which is always desirable for any SoC design. Moreover, the PSR in a self-biased BGR can be improved by using cascode current mirrors [3] or symmetric biasing of both the branches [5].

In this paper an improved self-biased based bandgap reference circuit has been proposed which further lowers the area and power of the reference circuit while pre-serving the temperature coefficient performance. The improved circuit generates the reference voltage without using the separate reference-voltage branch as in the tradi-tional self-biased BGR.
This paper is organized as follows: Section 2 describes the proposed architecture of the BGR along with its design procedure and circuit implementation. Simulation and measurement results are presented in Section 3, followed by a conclusion in Section 4.

## 2  Proposed Bandgap Reference
Fig. 2 shows the core part of the proposed bandgap reference circuit. As evident from the figure, this modified circuit avoids a bi-polar device in the reference branch. Here the BJT Q_2 used for a dual purpose; firstly, it helps for generating a PTAT voltage across resistor R_1 and secondly, voltage across this adds with voltage across R_2 for generating reference voltage (V_REF) at the output node. This elegant modification in the traditional self-biased current mirror based BGR provides some great advantages particularly in power consumption and silicon area of the core circuit. These advantages are:

- Since we eliminate the standard voltage-reference branch, the bi-polar device area reduces by approximately 20% and the PMOS current-mirror area reduces by approximately 30%. Note that, bi-polar devices and the current mirrors are a significant portion of the core BGR area.
- One-third of the total current is reduced in the core BGR and therefore one-third reduction in power consumption in the core BGR circuit as well.

The self-biased current mirror uses two P-MOS transistor 〖MP〗_1, 〖MP〗_2 and two N-MOS transistor 〖MN〗_1, 〖MN〗_2. These four transistor forms the self-biased feedback loop which makes the node voltages at ‘A’ and ‘B’ equal. The second branch of the circuit uses a single bi-polar device Q_2, which produces a CTAT voltage V_BE2  across the BJT Q_2, whereas, in the first branch, four parallel BJTs are connected with a resistor R_1 in series. As both the node voltages at ‘A’ and ‘B’ are the same and current flowing through both the BJTs are same, a PTAT voltage 〖dV〗_BE produced across resistor R_1. 

〖dV〗_BE=V_BE2-V_BE1 ------------------------------------------(2)

Where V_BE1 is voltage across the four parallel BJTs Q_1. 
As V_BE2 is a CTAT voltage and 〖dV〗_BE is a PTAT voltage, so the addition of CTAT voltage with some appropriate constant multiplication of the PTAT voltage will generate a reference voltage which will be zero temperature coefficients at a reference temperature.
The power-supply rejection (PSR) performance does not change significantly from the traditional self-biased BGR.  The PSR can be improved by using cascode current mirrors [3] or symmetric biasing of both the branches [5]. Our proposed integration of reference branch will also work with symmetric biasing as shown in [5].

### 2.1  Design Procedure of Improved BGR
In this section, the expressions to calculate the resistance values of the core BGR circuit for a current value will be shown. For a low power BGR, Q_1 and Q_2 were each biased with 1µA. Given the bias current,  R_1 can be expressed as:

R_1=V_T.ln⁡(4)/I_1 ---------------------------------------------(3)

<p align="center">
  <img src="/images/FIG2_BGR.png">
  Fig. 2. Core part of the proposed self-biased current mirror based BGR.
</p>

Where V_T is the thermal voltage of the semiconductor and its value at room temperature is approximately 25.8 mV. Applying the values of V_T and I_1 in equation (3), R_1 evaluates to 35.76 kΩ.
The reference voltage can be calculated by combining the voltage across the BJT Q_2 (CTAT in nature) and the voltage across the resistor R_2 (PTAT in nature) as;

V_REF=V_BE2+ V_R2 --------------------------------------------(4)

Where, V_R2 is the PTAT voltage across the resistor R_2 and can be expressed as:

V_R2=V_T.R_2/R_1 .ln⁡(4) --------------------------------------(5)

Equation (4) can be rewritten as;

V_REF=V_BE2+α.V_T --------------------------------------------(6)

Where, α= R_2/R_1 .ln⁡(4)   is a constant.

For calculating zero temperature coefficient reference voltage at the reference temperature, the derivative of V_REF should be zero.

(∂V_REF)/∂T=  ∂(V_BE2+α.V_T )/∂T=0 ---------------------------(7)

Using  (∂V_BE2)/∂T=-1.6 mV/ºC  and  (∂V_T)/∂T=85 µV/ºC [4] in equation (7), α evaluates to 18.82. 

Now we can calculate V_REF value from the equation (6) as:

V_REF=0.67V+18.82× 25.8 µV=1.155 V

For this modified architecture the current flowing through the resistor R_2 is half of that current flowing in the resistor R_1. So, the constant α for this circuit will be;

α= R_2/〖2R〗_1 .ln⁡(4) ---------------------------------------(8)

Applying α and R_1 values in equation (8), R_2 evaluates to 971 kΩ.

<p align="center">
  <img src="/images/FIG3_BGR.png">
  Fig. 3. Complete schematic diagram of the proposed BGR circuit.
</p>

### 2.2  Implementation of Improved BGR
Fig. 3 shows the complete implementation of the proposed BGR. 〖MP〗_(1-3), 〖MN〗_(1-2), R_(1-2), and Q_(1-2) forms the core part of the bandgap and the value of the resistors are calculated in the previous sub-section. 〖MPS〗_(1-2) and 〖MNS〗_1 form the startup circuitry since there are two stable states. 〖MP〗_B transistors are the PTAT current sources for biasing internal circuits. 〖MN〗_(1-2) are biased in the deep-sub-threshold (weak inversion) region to provide the maximum g_m⁄I_D  for a given bias current [6], which ensures the voltages of node ‘A’ and ‘B’ are only offset by the V_T mismatch of the 〖MN〗_(1-2) and the systematic offset of I_1 and I_2. Typically, (g_m⁄I_D )>20 ensures deep-sub-threshold operation. Please note that this offset is similar to an offset in an Op-Amp based BGR where the input referred offset of the Op-Amp is dominated by the V_T mismatch of the input pair of the differential amplifier which also biased in deep-subthreshold region for low-power application.

<p align="center">
  <img src="/images/FIG4_BGR.png">
  Fig. 4. Implementation of the 5-bit trimmable resistor R2.
</p>

PMOS current mirrors 〖MP〗_(1-3) and 〖MP〗_B are biased in the saturation region where g_m⁄I_D  is typically less than 10 [6], to ensure the minimum systematic offset in I_1 and I_2. As mentioned before, this systematic offset can be minimized by using cascode current mirrors [3] or symmetric biasing of both the branches [5]. The unit sizes of Q_1 and Q_2 are chosen to be the minimum allowable in the implemented technology and the ratio between them is chosen such that the area of Q_(1-2) and R_(1-2) is minimized. For the implemented technology, the BJT ratio 4:1 was found to be optimum. A high-sheet-rho poly resistor (R_sheet=3.76 kΩ/sq) was chosen to minimize the resistor area. In order to trim the output temperature coefficient (TC) of the BGR after fabrication, R_2 is a 5-bit programmable resistor is used as shown in Fig. 4, which is programmed through an Inter-IC Communication (I2C) protocol with a range of 890-940kΩ. Each of the programmable resistor in R_2 is made of series-parallel combination of unit resistors of 20kΩ. R_1 is also constructed from combination of same unit resistors so they can be matched in layout with R_2. During startup, 〖MPS〗_2 ensures that the current mirror is pulled out of the zero-V_gs state and once the circuit is operating normally (V_REF≈ 1.155), the voltage drop across 〖MNS〗_1 should be high enough that it shuts OFF 〖MPS〗_2. 〖MNS〗_1 needs to be sized such that there is no leakage current during normal operation. 〖MPS〗_1 provides a trickle current for 〖MNS〗_1 and 〖MNS〗_1 is sized with a very long length transistor to provide a large voltage drop for the minimum amount of current. For layout, special care is taken to match 〖MP〗_(1-3), 〖MN〗_(1-2), R_(1-2), and Q_(1-2) which affects the TC directly.

## 3  Simulation and Measurement Result
Fig. 5 and Fig. 6 shows the simulation results.

<p align="center">
  <img src="/images/FIG5_BGR.PNG">
  
  Fig. 5. Simulation result: VREF-vs-Temperature (Tempco)
</p>

<p align="center">
  <img src="/images/FIG6_BGR.png">

  Fig. 6. Simulation result: VREF versus Temperature for different R2 trims
</p>


### 3.1  Simulation Result
The improved self-biased bandgap reference has been simulated with a commercially available Spectre simulator using the Process Design Kit (PDK) from the foundry. The first order temperature drift performance is simulated over the entire temperature range of -40ºC to 125ºC. A simulated reference voltage (V_REF) versus temperature curve is shown in Fig. 5. The calculated temperature coefficient (TC) from the figure is 6.3 ppm/ºC. Fig. 6, shows the parametric plot of VREF  versus temperature at all 32 (5-bit) trimming resistance values. The simulated PSR performance at room temperature for the improved BGR circuit is about 40 dB at DC and 35 dB at 1 kHz. The noise performance at room temperature is 4.34 µV/√Hz at 10 Hz and 1.47 µV/√Hz at 100Hz which is dominated by the flicker noise of current-mirrors, 〖MN〗_(1-2) (46%) and 〖MP〗_(1-3) (52%). The simulated average quiescent current is about 2 µA over the temperature range of -40ºC to 125ºC. Table 1 summarizes the simulation parameters and their corresponding simulated values.

<p align="center">
  <img src="/images/TABLE1_BGR.png">
  Table 1. Summary of the Simulation Results.
</p>

<p align="center">
  <img src="/images/FIG7_BGR.png">
  Fig. 7. Chip micrograph and Layout of proposed BGR.
</p>

### 3.2	Test Setup and Measurement Results
This work has been fabricated in a commercially available 0.6µm CMOS technology. The proposed work has been integrated to provide a bias voltage to other blocks inside the chip. Fig. 7 shows the chip micrograph with highlighting the proposed BGR and its corresponding layout view. The whole BGR consumes 0.018 mm2 of silicon area inside the chip.

At the time of this writing, ability to do a full temperature characterization using an environmental chamber along with R_2 trimming through I^2 C was unavailable. A functional test of the fabricated BGR was done using the test setup as shown in Fig. 8 with the R_2 set to the default value. For the functional test, the packaged silicon chip is mounted on a temporary prototype board to test the functionality. We used a buffer (OP-90) at the output of the chip to avoid loading from the low-impedance measurement device. A hot air stream was used to heat the device to temperatures between 25ºC to 100ºC from the top side of the chip.

<p align="center">
  <img src="/images/FIG8_BGR.png">
  
  Fig. 8. Test Setup for the functional verification of the fabricated BGR
</p>

The temperature was changed by changing the distance between the source of the hot air stream and the device. The output of the BGR was measured using high preci-sion (6-1/2 digit) voltage meter (Keysight 34461A). After each temperature value settled, the temperature of the device was measured using a mounted laser-guided infrared thermometer. The device was powered using a programmable power supply (Keysight E3631A).

<p align="center">
  <img src="/images/FIG9_BGR.png">
  Fig. 9. Temperature Coefficient (tempco) plot.
</p>

<p align="center">
  <img src="/images/FIG10_BGR.png">
  Fig. 10. Line Regulation plot.
</p>

Fig. 9 shows the measurement result of output voltage versus temperature. As seen from the result, the untrimmed temperature coefficient is strongly PTAT in nature (115 ppm/ºC). Some of the random mismatch pairs that could contribute to this are 〖MP〗_(2-3), R_1⁄R_2  ratio, Q_(1-2) and 〖MN〗_(1-2) as well. In simulation, when R_2 is increased by 6.5% and V_T offset value of σV_T/√A is applied between 〖MP〗_(2-3) and 〖MN〗_(1-2) the simulation results match the test result as shown in Fig. 9.
For the same offsets added as for the tempco simulation, the line regulation in both simulations and measurements match closely showing a line regulation of 16 mV⁄V as shown in Fig 10. On availability of an environmental chamber, we will be able to get to the root of the tempco response by doing accurate temperature characterization for different R_2 trim value.

<p align="center">
  <img src="/images/TABLE2_BGR.png">
  Table-2. Comparison of core performance specification with previously published work.
</p>

## 4	Conclusion
In this paper, 	a self-biased based BGR was improved for area and power by eliminating the reference-voltage branch and integrating it in the main core without compromising temperature drift performance. By using the CTAT voltage in the core of the BGR to generate the reference voltage (V_REF), the power consumption of the core and area of the BJTs reduces by 33% and 20% respectively. The BGR is implemented in a 0.6-µm CMOS process with an area of 0.018 mm^2 that includes the core bandgap and bias currents. This architecture greatly simplifies the design complexity with a temperature coefficient of 6.3 ppm/ºC for a temperature range of -40ºC to 125ºC from simulation. The simulated PSR is 35 dB at 1kHz which can be improved by using the cascode self-biased current mirror. This architecture gives a spot noise of 4.34(µV)⁄√Hz dominated by the flicker noise of NMOS and PMOS current-mirrors. The flicker noise can be reduced by increasing the area of those devices or chopping the current mirror. Table-2 shows a comparison of the core performances with previously published work.

## References
1.	R. Widlar,: New developments in IC voltage regulators. 1970 IEEE International Solid-State Circuits Conference. Digest of Technical Papers, 1970, vol. XIII, pp. 158–159
2.	Y. Ji, C. Jeon, H. Son, B. Kim, H. Park, and J. Sim,: “5.8 A 9.3nW all-in-one bandgap volt-age and current reference circuit,” in 2017 IEEE International Solid-State Circuits Confer-ence (ISSCC), 2017, pp. 100–101
3.	W. Wu, W. Zhiping, and Z. Yongxue, “An Improved CMOS Bandgap Reference with Self-biased Cascoded Current Mirrors,” in 2007 IEEE Conference on Electron Devices and Sol-id-State Circuits, 2007, pp. 945–948.
4.	P. E. Allen and D. R. Holberg,: CMOS Analog Circuit Design. OUP USA (2012)
5.	Y. Lam and W. Ki, “CMOS Bandgap References With Self-Biased Symmetrically Matched Current–Voltage Mirror and Extension of Sub-1-V Design,” IEEE Transactions on Very Large Scale Integration (VLSI) Systems, vol. 18, no. 6, pp. 857–865, Jun. 2010.
6.	R. R. Harrison and C. Charles, “A low-power low-noise CMOS amplifier for neural record-ing applications,” IEEE Journal of Solid-State Circuits, vol. 38, no. 6, pp. 958–965, Jun. 2003.

# I2C
