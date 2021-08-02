# PHASE-LOCKED LOOP IC DESIGN USING SKY130 PDK

![PLL-Workshop-Banner_efabless](https://user-images.githubusercontent.com/88243788/127843029-c984bcd3-031e-4d7c-9a2b-687ffbf2afe2.png)


# DESCRIPTION:  
In this era of information, analog domain has become a challenging and an important part of IC design. Almost all electronic systems are susceptible to noise and mismatch. PLL is a an important element that is found in radio, telecommunications, oscillators among others. In this regard, a 2-day workshop on Phase-locked loop (PLL) IC design by VSD-IAT was conducted. The workshop covered the basic understanding of all the different blocks that make up a PLL. Using open-source tools such as Ngspice, Magic and Google-Skywater 130nm PDK, a 8x PLL clock multiplier IC was designed. Both pre-layout and post-layout simulations were carried out and detailed analysis was done to get an intuitive understanding of VLSI design flow (starts from device level to tapeout stage). Basic understanding of electric circuits is beneficial for a beginner.

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

However, one main issue with above configuration is the presence of Dead zone. It refers to the smallest difference in phase or frequency that the PFD is able to measure accurately without affecting the stability of the system. Any logic gate implemented in real life will have some delay associate with it. This delay makes the presence of Dead zone inevitable for PFD block.

## Charge Pump (CP):
The role of a charge pump in PLL is to convert the difference in phase/frequency measured digitally into an analog signal that can be used to control the voltage-controlled oscillator (VCO). This is achieved using current steering circuit. This circuit steers/directs the current flow from supply to output or output to ground depending on the UP and DOWN signal that is provided. The charge pump circuit gives an output voltage which corresponds to the average active time of UP and DOWN signals.

![image](https://user-images.githubusercontent.com/88243788/127896005-6398402a-e28e-4703-8d1f-6ccfa52c9936.png)

One main issue with the above transistor level design is that when the up and down transistors are OFF (Mpcsr and Mncsr), there is still some current flowing through them in form of leakage which may continue to charge the load capacitor. This is referred to as Charge leakage.

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
3) Google-Skywater 130 PDK- for specifications of the transistors
4) Caravel- a medium to ensure that the designed IP is tapeout ready




# ACKNOWLEDGEMENT:  
1) I would like to thank Mr. Kunal Ghosh (Co-founder VSD), for providing me an opportunity to partake in this workshop and understand VLSI design flow process both theoritically and practically.
2) I would also like to thank Ms. Lakshmi S. for providing access to spice files and guidance throughout for the implementation of PLL system. 
3) Lastly, I appreciate the TAs for clarifying my doubts during lab sessions without which this work would not have been made presentable.
