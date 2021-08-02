# PHASE-LOCKED LOOP IC DESIGN USING SKY130 PDK

![PLL-Workshop-Banner_efabless](https://user-images.githubusercontent.com/88243788/127843029-c984bcd3-031e-4d7c-9a2b-687ffbf2afe2.png)


# DESCRIPTION:  
In this era of information, analog domain has become a challenging and an important part of IC design. Almost all electronic systems are susceptible to noise and mismatch. PLL is a an important element that is found in radio, telecommunications, oscillators among others. In this regard, a 2-day workshop on Phase-locked loop (PLL) IC design by VSD-IAT was conducted from 31st July, 2021 to 1st August, 2021. The workshop covered the basic understanding of all the different blocks that make up a PLL. Using open-source tools such as Ngspice, Magic and Google-Skywater 130nm PDK, a 8x PLL clock multiplier IC was designed. Both pre-layout and post-layout simulations were carried out and detailed analysis was done to get an intuitive understanding of VLSI design flow (starts from device level to tapeout stage). Basic understanding of electric circuits is beneficial for a beginner.

# CONTENTS:
- Day 1:  
  - PLL Theory  
  - Phase-Frequency detector  
  - Charge pump  
  - Loop filter  
  - Voltage-controlled oscillator  
  - Frequency dividor  
  - Tool setup  
  - Development Flow  
  - PLL Specifications  
- Day 2: Simulations  
  - Pre-Layout Simulations  
    - Phase-Frequency detector  
    - Charge pump  
    - Voltage-controlled oscillator  
    - Frequency divider  
    - Troubleshooting Steps  
  - Layout Design  
    - Phase-Frequency detector  
    - Charge pump  
    - Voltage-controlled oscillator  
    - Frequency divider  
    - MUX  
    - PLL System  
  - Post-Layout Simulations  
    - Phase-Frequency detector  
    - Charge pump  
    - Voltage-controlled oscillator  
    - Frequency divider  
    - PLL System  
  - Tapeout  
  - Acknowledgement
  

# Day 1:  
## PLL Theory:  
A phase-locked loop (PLL) is a control system that compares the phase of input reference signal and output desired signal through a feedback loop. The goal of PLL system is to generate a very precise clock signal without frequency or phase noise and at the same time have the flexibility of running at the desired frequency.  

The basic block diagram of PLL system is shown below:  

![image](https://user-images.githubusercontent.com/88243788/127889747-a5af5d86-3a1b-4aa4-b11e-f2ac80665eb0.png)

Usually, to generate clock signal, either Quartz crystals or Voltage-controlled oscillators (VCOs) is used depending on the application. VCOs offer good flexibility and better control frequency with input voltage. They are mostly used for on-chip applications. However, they are susceptible to noise and/or fluctuations in their phase. This is refered to as Phase noise. On the other hand, Quartz crystals provide superior spectral purity and no unwanted frequency.

Note: Spectral purity refers to a frequency spectrum having least amount of unwanted frequencies.  

![image](https://user-images.githubusercontent.com/88243788/127888401-0b01a658-bb59-4ba7-be69-f6ab26f13692.png)

We shall now look into each block of the PLL system and understand its functionality:

## Phase-Frequency Detector (PFD):  
It is used to to compare the frequency divided by 8 signal and the reference input signal. Using XOR function, one cannot distinguish between leading and lagging nature of input or output. To this extent, one introduces 2 additional signals: UP and DOWN that tell when the output signal needs to be sped up or slowed down respectively. Also, the width of output signal pulse tells to what extent the phase difference needs to be increased or decreased.  

![image](https://user-images.githubusercontent.com/88243788/127891327-c897a93b-e2b3-46ae-99ab-5783b3eff452.png)

From the figure above, The DOWN signal becomes ACTIVE when falling edge of output is detected till falling edge of reference signal. Similarly, the UP signal is detected when falling edge of reference signal is detected till falling edge of output signal. 

![image](https://user-images.githubusercontent.com/88243788/127891609-75fecf45-dc62-40f2-b909-6e1b224adb05.png)

Similarly, for signals of different frequency, if the output signal frequency is higher than the reference signal, DOWN signal is ACTIVE. This indicates that we must slow down the output. Likewise, if the output signal frequency is lower, UP signal becomes ACTIVE. Hence, we must speed up the output. In this way, these 2 signals (UP and DOWN) help capture the frequency/phase difference between the reference and the output. This is also shown below in the form of a state diagram:

![image](https://user-images.githubusercontent.com/88243788/127892100-c0a176de-91a9-4af1-845c-d094f4830844.png)

To implement PFD, since we have to detect falling edges of two different signals, one uses 2 edge-triggered flip-flops as shown below. Moreover from the understanding established above in regards to UP and DOWN signals, one would require AND gate as well.

![image](https://user-images.githubusercontent.com/88243788/127892546-30bed6be-2f82-4080-9808-4f073514d9a0.png)

However, one main issue with above configuration is the presence of Dead zone. It refers to the smallest difference in phase or frequency that the PFD is able to measure accurately without affecting the stability of the system. Any logic gate implemented in real life will have some delay associate with it. This delay makes the presence of dead zone inevitable for PFD block.

## Charge Pump (CP):
The role of a charge pump in PLL is to convert the difference in phase/frequency measured digitally into an analog signal that can be used to control the voltage-controlled oscillator (VCO). This is achieved using current steering circuit. This circuit steers/directs the current flow from supply to output or output to ground depending on the UP and DOWN signal that is provided. The charge pump circuit gives an output voltage which corresponds to the average active time of UP and DOWN signals.

![image](https://user-images.githubusercontent.com/88243788/127896005-6398402a-e28e-4703-8d1f-6ccfa52c9936.png)

One main issue with the above transistor level design is that when the up and down transistors are OFF (Mpcsr and Mncsr), there is still some current flowing through them in form of leakage which may continue to charge the load capacitor. This is referred to as charge leakage. To tackle this issue, an additional set of transistors as shown below:

![image](https://user-images.githubusercontent.com/88243788/127900524-7a1f7ee7-49f6-4fd2-a1f2-7b94d90eae73.png)

## Loop Filter:
Recall, the output capacitor of the CP circuit helps smoothens the output. However, there are still fluctuations caused due to rise and fall of UP and DOWN signals. To mitigate this, one may use a low-pass filter (LPF) at the output instead of just a capacitor. This smoothens out any high-frequency fluctuations in the output. It also helps in stabilizing the PLL system. Recall, a loop filter is same as adding a zero to the frequency domain of the PLL which enable the system to be stable. Without LPF, the PLL cannot lock and mimic the reference signal.  

![image](https://user-images.githubusercontent.com/88243788/127896041-afa78acd-cdf5-4fcb-8f7c-ad5bcc18c0e2.png)

As a thumb rule, Cx ~= C/10 and the loop filter bandwidth ~= 0.1*(highest output frequency intend to give for a PLL) for stability of the system.

## Voltage-controlled Oscillator (VCO):
A series combination of odd number of inverters is used to make a ring oscillator. This offers an output whose period is twice the total delay of all the inverters. However, to make it flexible, a current starving mechanism is used to control the oscillating frequency. Two current sources are used as supplies for the ring oscillator. This enable the control on its frequency based on an input-controlled voltage. The VCO is to be designed such that the range of output frequencies we want for the PLL is within the range of frequencies that this VCO can produce properly.

![image](https://user-images.githubusercontent.com/88243788/127896919-f1616be2-f0b5-48fe-9bab-342c4374d62c.png)

## Frequency divider (FD):
The output of a Toggle flip-flop will be half the frequency of the input. TFF is implemented using DFF as shown below. Here, the faded green signal represents the input reference signal.

![image](https://user-images.githubusercontent.com/88243788/127897408-95fc3582-edb2-40d2-b00f-87626606c518.png)

Now, to implement a 8x clock multiplier, then we can add 3 TFFs to get the desired result.

Before, proceeding further, we shall look at some common terminologies used:

1) Lock Range: The range of frequencies the PLL is able to follow input frequency variations once locked. It is mainly defined by the range of frequencies VCO can produce and is limited by the dead zone of the PFD block.
2) Capture Range: It refers to the range of frequencies for which the PLL is able to lock-in when starting from an unlocked condition. This is usually smaller than the lock range and depends on the loop filter bandwidth.
3) Settling Time: It is the time within which the PLL is able to lock-in from an unlocked condition. This depends on how quickly the charge pump rises to a stable value.

## Tool Setup:
Following open-source tools were needed for PLL IC design:
1) Ngspice- for transistor level spice simulations
2) Magic- for layout design and parasitic extraction
3) Google-Skywater 130nm PDK- for primitive libraries of the transistors to be used during simulations using Ngspice
4) Caravel- a medium to ensure that the designed IP is tapeout ready

Note: A PDK (process-development kit) is a collection of libraries, standard cells and other files which are required for desiging an IC.

To install:  
Ngspice-> sudo apt-install ngspice (from here we fetch Sky130nm library file)  
Magic-> We need to git clone the source code from opendesigncircuit.com. Then we install the dependencies followed by Magic tool. This is followed by compilation using following command: sudo make install  

To use/run a circuit file using:  
Ngspice-> ngsice <circuit_file_name>  
Magic-> magic -T <Technology_file_from_PDK> <the_layout_file_to_open>  

## Development Flow:
1) SPICE-level circuit development
2) Pre-layout simulation
3) Layout development
4) Parasitic extraction
5) Post-layout simulation (more accurate than pre-layout simulations)

It is often the case that after each step, one needs to make necessary modifications to the circuit

## PLL Specifications:

1) Corner- 'TT' (Typical-Typical)
2) Supply voltage= 1.8 V
3) Temperature = Room temperature
4) VCO mode and PLL mode
5) Input Fmin= 5MHz , Fmax= 12.5MHz
6) Multiplier = 8x
7) Jitter (RMS) < 20ns (measure of phase noise)
8) Duty cycle = 50%

# Day 2: Simulations
To get started with simulations, it is necessary to create a SPICE file. It is a text file with .cir extension. We will create SPICE files for each of the building blocks of PLL system before proceeding with simulations.

## Pre-Layout Simulations:
To run pre-layout simulations the command is:  
ngspice <file_name>.cir  

### Phase-Frequency detector (PFD):

![image](https://user-images.githubusercontent.com/88243788/127902238-124764cb-b3b6-4c0b-b7a2-c239e0f3db27.png)

In above graph, the different colors represent:  
Red: Clock 1  
Blue: Clock 2  
Orange: UP Signal  
Green: DOWN Signal

### Charge-Pump (CP):

![image](https://user-images.githubusercontent.com/88243788/127902497-9be94f90-ef67-423c-9b82-ae9b3491d84e.png)

Clearly, slope is 40 V/s.

### Voltage-Controlled Oscillator (VCO):

![image](https://user-images.githubusercontent.com/88243788/127902964-f830835f-1f20-4517-a464-c4bd70ccfd93.png)

Here, the control voltage is represented byy Red color while output clock is shown by blue color.  

### Frequency Divider (FD):

![image](https://user-images.githubusercontent.com/88243788/127903180-c1528b85-53fb-4aeb-94c8-f1ea0ab2db38.png)

Here, input clock is shown in blue and output clock is shown in red color.  

### Troubleshooting Steps:  
Usually, it so happens that the output doesn't properly lock (or mimic the input reference) due to some errors in circuit designing. In such a case, following sequence of steps are usually followed:  

- Identify the issues that might be occurring in the simulations or circuit.
- Try to debug individual components and confirm their functionality before simulating the entire circuit.  
- If the waveforms are flat or simulation is crashing, then:  
  - Check whether all connectivities have been done properly.  
  - Check the spelling of each net, model name, case-sensitive issues, parameter value assignment.  
  - Check if any pin is left unconnected.  
- In case of correct waveforms but improper mimicking, then:  
  - Find the range of frequencies for which VCO is working properly. The required output frequency range should lie within VCOs frequency range.  
  - Check if the PFD block is able to detect minute (small) changes in the phase difference properly; else it may lead to phase noise in the system. This may cause the PLL system to become unstable.  
  - Rate at which the output of charge-pump block is charging/discharging. In case of fluctuations in the output voltage, one may need to re-size the transistors appropriately to get a smooth response.  
  - Check whether output is charging in case of no input (i.e. input is 0). If yes, then charge leakage is the main issue.  
  - Adjust the loop filter values (while ensuring the stability of the system) to get desired response.  

## Layout Design:
Color Specifications used:
1) p-diffusion - orange  
2) n-diffusion - green  
3) Polysilicon - red  
4) n-well - dashed lines  
5) Metal1 layer - purple  
6) Local interconnect layer - blue

While making connections, we use via to connect different metal layers and and interconnect layer between two transistors.  
We shall now look at the layouts of the different blocks of PLL system.  

### Phase-Frequency Detector:

![image](https://user-images.githubusercontent.com/88243788/127904631-03b2f06a-de8b-4565-b223-1ad6cb00faeb.png)

### Charge-Pump (CP):

![image](https://user-images.githubusercontent.com/88243788/127904908-ad8e885d-35c9-4d87-9017-3c343302e055.png)

### Voltage-Controlled Oscillator (VCO)

![image](https://user-images.githubusercontent.com/88243788/127904974-5b2de1a3-4d4f-4d36-ae74-6e4751f0a0f1.png)

### Frequency Divider (FD):

![image](https://user-images.githubusercontent.com/88243788/127905002-c0d8bde5-704b-48da-9dbe-4c7167128d07.png)

### MUX:

![image](https://user-images.githubusercontent.com/88243788/127905077-442151cb-5581-4c14-b11f-97dee7f13110.png)

### PLL System:

![image](https://user-images.githubusercontent.com/88243788/127905043-03babaee-b073-429b-8a32-39d3a1852a98.png)

## Post-Layout Simulations:
To run post-layout simulations, the command is:  
ngspice <file_name>.cir  
Example: ngspice PFD_PostLay.cir  
Here, we conduct post-layout simulations for all the blocks of PLL system.

### Phase-Frequency Detector (PFD):

Below waveforms represent the case when UP signal is ACTIVE:  

![image](https://user-images.githubusercontent.com/88243788/127905677-0984bb9b-f8f3-4181-840b-52fc2c904c6f.png)

Whereas, below is the case, when DOWN signal is ACTIVE:

![image](https://user-images.githubusercontent.com/88243788/127905722-44a2248c-f66f-4c34-bb43-6df2e1b8960d.png)

In both above cases, color terminology used is:  
Clock1-> red  
Clock2-> blue  
UP signal-> orange  
DOWN signal-> green  

### Charge-Pump (CP):

Below is the case when UP signal is ACTIVE (= '1'):  

![image](https://user-images.githubusercontent.com/88243788/127906205-a88e1b79-4cc3-44d5-9cc6-b4e76503f2f4.png)

Below is the case when DOWN signal is ACTIVE (= '1'):  

![image](https://user-images.githubusercontent.com/88243788/127906322-ad405a49-15a2-49d5-8b39-50db6afa2043.png)

However, recall one of the main limitations of CP is charge leakage which causes output to charge in absence of input. This is demonstrated in below waveform.  

![image](https://user-images.githubusercontent.com/88243788/127906520-ad0cab84-724e-488c-878c-786f28b54915.png)

In all the waveforms above, the color terminology is as follows:  
UP signal-> red  
DOWN signal-> blue  
Output voltage (of CP block)-> orange  

### Voltage-Controlled Oscillator (VCO):

![image](https://user-images.githubusercontent.com/88243788/127906868-c4d1cfaf-39a1-4ead-80d5-a003558444ca.png)

Here, red represents the output clock whereas blue represents the control voltage.  

### Frequency-Divider (FD):

![image](https://user-images.githubusercontent.com/88243788/127907020-6677865c-a0de-4dcc-a220-2685588633d4.png)

Here, red represents the input clock and blue represents the output clock.  

### PLL System:
Here, we have shown simulations for 2 cases for a 8x clock multiplier PLL system:  

1) When input frequency is 5MHz, then output frequency is 40MHz:  

![image](https://user-images.githubusercontent.com/88243788/127910751-2a910c02-9b27-40f8-9b6b-def92827186c.png)

2) When input frequency is 12.5MHz, then output frequency is 100MHz:  

![image](https://user-images.githubusercontent.com/88243788/127910780-3e9a32e6-318f-4789-8d2b-20590250e020.png)

In both the above cases, the color terminology used is:  
Input reference-> red  
Output clock-> blue  
UP signal-> brown  
DOWN signal-> yellow  
Charge-pump output-> purple  



# ACKNOWLEDGEMENT:  
1) I would like to thank Mr. Kunal Ghosh (Co-founder VSD), for providing me an opportunity to partake in this workshop and understand VLSI design flow process both theoritically and practically.
2) I would also like to thank Ms. Lakshmi S. for providing access to spice files and guidance throughout for the implementation of PLL system. 
3) Lastly, I appreciate the TAs for clarifying my doubts during lab sessions without which this work would not have been made presentable.
