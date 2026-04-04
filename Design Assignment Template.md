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
2. ADC resolution: 12 bits
3. Maximum allowed event-to-response latency: 2ms

With these constraints in mind, we first find the minimum sampling rate. Here, we apply Nyquist-Shannon Sampling Theorem, which states bandlimited frequencys with
$f_{max}$ can be reconstructed perfectly where: $$f_s \geq 2 f_{max}$$, and need anti-aliasing filters to ensure $$\frac{f_s}{2} \geq f_{max}$$. Apply rule-of-thumb: sample **2-10x** $f_{max}$ for margin & filter roll-off.

### 2. DMA-Based Data Acquisition

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
