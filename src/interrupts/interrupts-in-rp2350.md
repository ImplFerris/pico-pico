# Interrupts in the RP2350

In the previous chapter, we looked at what interrupts are and the role of the NVIC. Now, lets look at which interrupts are actually available on the RP2350.

Interrupts fall into two groups: system exceptions and external interrupts.

System exceptions are defined by the CPU architecture itself. These include reset, fault handlers, and the system timer. They behave the same way across most Cortex-M chips.

External interrupts come from peripherals on the RP2350. Each peripheral that can generate an interrupt has an IRQ number and a vector name. These are the names you will see in code.

The table below shows the external interrupts on the RP2350, numbered from 0 to 51. They cover common peripherals such as timers, GPIO, DMA, and communication interfaces like I2C, SPI, and UART.

You do not need to memorize this table. Its purpose is to help you recognize where names like `I2C0_IRQ` or `UART0_IRQ` come from when you see them in examples or documentation.

The full details are in the [RP2350 datasheet, section 3.2](https://pip-assets.raspberrypi.com/categories/1214-rp2350/documents/RP-008373-DS-2-rp2350-datasheet.pdf?disposition=inline#page=83) on page 82.

In the next chapter, we will see how Embassy uses these interrupts without requiring you to write interrupt handlers manually.

<div class="interrupt-container">
<header>
    <h1>RP2350 External Interrupts</h1>
</header>

> [!IMPORTANT]
> Some interrupt descriptions are simplified here for a beginner-friendly overview. For more accurate and detailed information, refer to the RP2350 datasheet.

<div class="interrupt-content">

<h2 class="interrupt-section-title">Timers</h2>
<p>Timer alarms used for delays and scheduling.</p>
<table>
<thead>
<tr><th>IRQ</th><th>Vector</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>0</td><td>TIMER0_IRQ_0</td><td>Timer 0 alarm interrupt</td></tr>
<tr><td>1</td><td>TIMER0_IRQ_1</td><td>Timer 0 alarm interrupt</td></tr>
<tr><td>2</td><td>TIMER0_IRQ_2</td><td>Timer 0 alarm interrupt</td></tr>
<tr><td>3</td><td>TIMER0_IRQ_3</td><td>Timer 0 alarm interrupt</td></tr>
<tr><td>4</td><td>TIMER1_IRQ_0</td><td>Timer 1 alarm interrupt</td></tr>
<tr><td>5</td><td>TIMER1_IRQ_1</td><td>Timer 1 alarm interrupt</td></tr>
<tr><td>6</td><td>TIMER1_IRQ_2</td><td>Timer 1 alarm interrupt</td></tr>
<tr><td>7</td><td>TIMER1_IRQ_3</td><td>Timer 1 alarm interrupt</td></tr>
</tbody>
</table>

<h2 class="interrupt-section-title">PWM</h2>
<p>PWM counter wrap events.</p>
<table>
<thead>
<tr><th>IRQ</th><th>Vector</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>8</td><td>PWM_IRQ_WRAP_0</td><td>PWM wrap interrupt</td></tr>
<tr><td>9</td><td>PWM_IRQ_WRAP_1</td><td>PWM wrap interrupt</td></tr>
</tbody>
</table>

<h2 class="interrupt-section-title">DMA</h2>
<p>DMA transfer events.</p>
<table>
<thead>
<tr><th>IRQ</th><th>Vector</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>10</td><td>DMA_IRQ_0</td><td>DMA transfer interrupt</td></tr>
<tr><td>11</td><td>DMA_IRQ_1</td><td>DMA transfer interrupt</td></tr>
<tr><td>12</td><td>DMA_IRQ_2</td><td>DMA transfer interrupt</td></tr>
<tr><td>13</td><td>DMA_IRQ_3</td><td>DMA transfer interrupt</td></tr>
</tbody>
</table>

<h2 class="interrupt-section-title">USB</h2>
<p>USB controller events.</p>
<table>
<thead>
<tr><th>IRQ</th><th>Vector</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>14</td><td>USBCTRL_IRQ</td><td>USB controller interrupt</td></tr>
</tbody>
</table>

<h2 class="interrupt-section-title">PIO</h2>
<p>PIO state machine events.</p>
<table>
<thead>
<tr><th>IRQ</th><th>Vector</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>15</td><td>PIO0_IRQ_0</td><td>PIO 0 interrupt</td></tr>
<tr><td>16</td><td>PIO0_IRQ_1</td><td>PIO 0 interrupt</td></tr>
<tr><td>17</td><td>PIO1_IRQ_0</td><td>PIO 1 interrupt</td></tr>
<tr><td>18</td><td>PIO1_IRQ_1</td><td>PIO 1 interrupt</td></tr>
<tr><td>19</td><td>PIO2_IRQ_0</td><td>PIO 2 interrupt</td></tr>
<tr><td>20</td><td>PIO2_IRQ_1</td><td>PIO 2 interrupt</td></tr>
</tbody>
</table>

<h2 class="interrupt-section-title">GPIO and Core I/O</h2>
<p>GPIO and core signaling events.</p>
<table>
<thead>
<tr><th>IRQ</th><th>Vector</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>21</td><td>IO_IRQ_BANK0</td><td>GPIO interrupt</td></tr>
<tr><td>22</td><td>IO_IRQ_BANK0_NS</td><td>GPIO interrupt</td></tr>
<tr><td>23</td><td>IO_IRQ_QSPI</td><td>QSPI GPIO interrupt</td></tr>
<tr><td>24</td><td>IO_IRQ_QSPI_NS</td><td>QSPI GPIO interrupt</td></tr>
<tr><td>25</td><td>SIO_IRQ_FIFO</td><td>Inter-core FIFO interrupt</td></tr>
<tr><td>26</td><td>SIO_IRQ_BELL</td><td>Inter-core doorbell interrupt</td></tr>
<tr><td>27</td><td>SIO_IRQ_FIFO_NS</td><td>Inter-core FIFO interrupt</td></tr>
<tr><td>28</td><td>SIO_IRQ_BELL_NS</td><td>Inter-core doorbell interrupt</td></tr>
<tr><td>29</td><td>SIO_IRQ_MTIMECMP</td><td>System timer interrupt</td></tr>
</tbody>
</table>

<h2 class="interrupt-section-title">Communication Peripherals</h2>
<p>Communication interface events.</p>
<table>
<thead>
<tr><th>IRQ</th><th>Vector</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>30</td><td>CLOCKS_IRQ</td><td>Clock system interrupt</td></tr>
<tr><td>31</td><td>SPI0_IRQ</td><td>SPI interrupt</td></tr>
<tr><td>32</td><td>SPI1_IRQ</td><td>SPI interrupt</td></tr>
<tr><td>33</td><td>UART0_IRQ</td><td>UART interrupt</td></tr>
<tr><td>34</td><td>UART1_IRQ</td><td>UART interrupt</td></tr>
<tr><td>35</td><td>ADC_IRQ_FIFO</td><td>ADC FIFO interrupt</td></tr>
<tr><td>36</td><td>I2C0_IRQ</td><td>I2C interrupt</td></tr>
<tr><td>37</td><td>I2C1_IRQ</td><td>I2C interrupt</td></tr>
</tbody>
</table>

<h2 class="interrupt-section-title">System and Power</h2>
<p>System and power management events.</p>
<table>
<thead>
<tr><th>IRQ</th><th>Vector</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>38</td><td>OTP_IRQ</td><td>OTP interrupt</td></tr>
<tr><td>39</td><td>TRNG_IRQ</td><td>Random number generator interrupt</td></tr>
<tr><td>40</td><td>Reserved</td><td>Reserved</td></tr>
<tr><td>41</td><td>Reserved</td><td>Reserved</td></tr>
<tr><td>42</td><td>PLL_SYS_IRQ</td><td>System PLL interrupt</td></tr>
<tr><td>43</td><td>PLL_USB_IRQ</td><td>USB PLL interrupt</td></tr>
<tr><td>44</td><td>POWMAN_IRQ_POW</td><td>Power manager interrupt</td></tr>
<tr><td>45</td><td>POWMAN_IRQ_TIMER</td><td>Power manager timer interrupt</td></tr>
</tbody>
</table>

<h2 class="interrupt-section-title">Software IRQs</h2>
<p>Interrupts that can be triggered by software.</p>
<table>
<thead>
<tr><th>IRQ</th><th>Vector</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td>46</td><td>SPAREIRQ_IRQ_0</td><td>Software interrupt</td></tr>
<tr><td>47</td><td>SPAREIRQ_IRQ_1</td><td>Software interrupt</td></tr>
<tr><td>48</td><td>SPAREIRQ_IRQ_2</td><td>Software interrupt</td></tr>
<tr><td>49</td><td>SPAREIRQ_IRQ_3</td><td>Software interrupt</td></tr>
<tr><td>50</td><td>SPAREIRQ_IRQ_4</td><td>Software interrupt</td></tr>
<tr><td>51</td><td>SPAREIRQ_IRQ_5</td><td>Software interrupt</td></tr>
</tbody>
</table>

</div>
</div>
