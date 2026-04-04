#ECE 315 – Design Assignment: Real-Time Data Acquisition 

<any text between angle brackets is informational and should be deleted in the final version of the report.>

## Introduction
<summarize the problem, objectives, and approach to this assignment>

## System Overview
<Write a brief overview of the system architecture and include a block diagram. An example is provide below. Make sure to identify and describe each node and component on the diagram. A table may be used for this>
A high-level deployment diagram for the envisioned system is shown in Figure 1.

![An example system deployment diagram using UML2](images/Block_Diagram_Example.png)
Figure 1 An example system deployment diagram using UML2

As mentioned in the introduction, the main function of the system is to control motor rotation direction based on a single input control signal. The name and purpose of each of the nodes and components shown in Figure 1 are summarize in Table 1 below.

Table 1 Description of the main system nodes and components
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>HW or FW</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Row 1, Cell 1</td>
      <td>Row 1, Cell 2</td>
      <td>Row 1, Cell 3</td>
    </tr>
    <tr>
      <td>Row 2, Cell 1</td>
      <td>Row 2, Cell 2</td>
      <td>Row 2, Cell 3</td>
    </tr>
    <tr>
      <td>Row 3, Cell 1</td>
      <td>Row 3, Cell 2</td>
      <td>Row 3, Cell 3</td>
    </tr>
  </tbody>
</table>

Each of the main design tasks related to the architecture are summarized in the following subsections.

## Design Tasks

### 1. ADC and Sampling Design
Recall design constraints for ADC and sampling:
1. Analogue input bandwidth: 0-2 kHz

**Part 1.1**
With these constraints in mind, we first find the minimum sampling rate. Here, we apply Nyquist-Shannon Sampling Theorem, which states bandlimited frequencys with
$f_{max}$ can be reconstructed perfectly where: $$f_s \geq 2 f_{max}$$, and need anti-aliasing filters to ensure $$\frac{f_s}{2} \geq f_{max}$$. Apply rule-of-thumb: sample **2-10x** $f_{max}$ for margin & filter roll-off.

* Thus, we find that $f_{max}$ is the max analogue input signal of 2 kHz
* A minimum rate (Nyquist) would be 4 kS/s (4 kHz), however we want to give ourselves some headroom and aliasing budget
* Multiply this value by 3, and we get the sampling rate of: $f_s = 6 kHz = 6 kS/s
* The new Nyquist Frequency is $$f_N \geq \frac{f_s}{2}$$, thus $$f_N = \frac{6 kS/s}{2} = 3 kS/s = 3 kHz
* Rate was choosen to help with design overhead (margin), reduce aliasing folding, use of anti-alias filter, keep system cost low

**Part 1.2**
We are then told the ADC samples 4 analogue channels in roun-robin mode, need to determine an effective per-channel sampling rate. We note that for the RP2040,
the ADC is set for muliple inputs and can utilize round-robin sampling. From the documentaiton, the RP2040 has only one ADC FIFO, so the sample from each source
(channel) are placed into the FIFO in an interleaved fashion, which must later be de-interleaved.

* To find the sample rate per channel, we use $$f_{per \space channel} = \frac{f_s}{M}$$ where M is the number of channels.
* Four channels to sample: $$f_{per \space channel} = \frac{6 kS/s}{4} = 1.5 kS/s$$.
* This is below the input range of 0-2 kHz for sampling, thus, we need to re-evaluate our total sampling rate.
* Set each channel to have sampling rate of 8 kS/s, thus: $$f_s = f_{per \space channel} \cdot M = 8 kS/s \cdot 4 = 32 kS/s.
* The new total Nyquist Frequency is $$f_N = \frac{f_s}{2} = \frac{24 kS/s}{2} = 16 kS/s$$.
* We now exceed the Nyquist frequency, which helps minimize the aliasing.
* Note that if we had kept the original sample frequencies, the Nyquist frequency is severly lowered, and thus the signal fidelity would be decreased. Fidelity
  is affected by the resolution (bit depth), time (sampling rate, jitter), noise (thermal), etc. Thus, by increaseing the sample rate for the per channel, the
  fidelity increases, and minimizes distortion.

**Part 1.3**
Anti-aliasing filters are almost always recommended for ADC sampling, pickup from stray signals (powerline frequency or even local radio stations!) may contain such frequencies higher than the Nyquist Frequency, and thus these frequencies may alias into the appropriate frequency range.

* We use filters to anti-alias sampling for ADCs, in our design we wouldnt necessarily need one, but we will implement one for good practice
* Add a low-pass filter (passes low frequencies, but attenuates the high frequencies) before the sampler and ADC
* Based on our calculations, the transition band occurs at the max input frequency and the Nyquist (per channel) rate: $$f_{band} = f_{N \space Channel} - f_{max}
  = 4 kS/s - 2 kS/s = 2 kS/s$$ for the bandwith range of filtering
* Thus all the frequencies from 0-2 kHz will pass, 2-4 kHz are attenuated, and anything above the Nyquist frequency are rejected
* Thus, with oversampling, low-pass filter filtering we exceed requirements (could also implement digital decimation after filtering)

### 2. DMA-Based Data Acquisition
Recall design constraints for ADC:
1. ADC resolution: 12 bits
2. Note that the faster the ADC samples, the fewer the number of bit out. N-bit ADC can quantize the input signal to $$2^N$$ levels.   
We also note that the digital output range is approximately $$[-2^{N-1}, 2^{N-1} \space volts]$$.
3. The RP2040 DMA features are: 12 independent DMA channels, separate read/write bus masters, transfer 8/16/32-bit words with one read+write per cycle, supports
   paced (TREQ), and 100s MB/s throughput.

**Part 2.1**
Reviewing the RP2040 data sheet, the ECE315 Lecture notes, we find that the ADC will continously be sampled, where the sample data is pushed to the ADC FIFO,
and then the DMA moves the data from the FIFO to a RAM buffer, and when a block is full the DMA signals to the CPU with an interrupt.

* The source for the DMA is the ADC FIFO register. From the datasheet the ADC_BASE is at *0x4004C00* with the ADC_FIFO at *0x4004C00C which is the source address
* When sending the data, the DMA will send this to a RAM buffer (array inside the RAM or the *block*), in this case we will use SRAM0 which will be the destination of the
  data at address *0x21000000*

**Part 2.2**

**Part 2.3**

**Part 2.4**

   
<Show a UML messaging diagram that illustrates the interaction between the ADC, DMA peripheral, and CPU>
The interaction between the ADC, DMA peripheral, and CPU is illustrated in the messaging diagram of Figure 2.

![An example messaging diagram](images/UML_Messaging_template.png)
Figure 2 An example messaging diagram

### 3. Interrupt and Processing Strategy

### 4. Motor Control

### 5. Feasibility and CPU Budget Analysis

### 6. System-Level Design Tradeoffs


## AI Usage Declaration

<Please indicate if you used an AI to help with this assignment and if so, what insights you gained; for example how it helped you understand this assignment>

## Resources

ECE315 Lecture Notes   
[1] https://www.ti.com/lit/an/slyt626/slyt626.pdf?ts=1775333427164&ref_url=https%253A%252F%252Fwww.google.com%252F   
[2] https://www.ni.com/en/shop/data-acquisition/measurement-fundamentals/analog-fundamentals/anti-aliasing-filters-and-their-usage-explained.html   
[3] https://www.youtube.com/watch?v=Sv_wMr9_Mwk
[4] https://forums.ni.com/t5/Multifunction-DAQ/what-do-you-mean-by-MS-s-or-KS-s-as-unit-of-frequency-I-m-used/td-p/925278   
[5] https://forums.ni.com/t5/Multifunction-DAQ/What-is-the-unit-kS-s-stand-for/m-p/4258344#M10295   
[6] https://pip-assets.raspberrypi.com/categories/814-rp2040/documents/RP-008371-DS-1-rp2040-datasheet.pdf?disposition=inline   

