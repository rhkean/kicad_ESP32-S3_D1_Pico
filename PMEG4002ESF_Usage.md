# Universal Diode Usage:  PMEG4002ESF

1. [x] reverse polarity protection for the TPS2121 power mux with 2 5V inputs. (we will revisit the voltage offset for the configuration resistors later)
    - GND &rarr; (K)PMEG4002ESF(A) &rarr; TPS2121-12
2. [x] reverse polarity indicator LED. this is a simple circuit before the TPS2121 that is wired in reverse:   
    - GND &rarr; 470R &rarr; LED &rarr; (A)PMEG4002ESF(K) &rarr; 5V
3. [x] TCAN Vsup input. VBat connection with TVS at connector, PMEG4002ESF in series to Vsup
    - $V_{BAT}$ &rarr; TVS &rarr; (A)PMEG4002ESF(K) &rarr; TCAN4550-14
4. [x] Vehicle, active low, +12V Dome light signal:
    - signal wire -> TVS -> (k)PMEG(A) -> 470R -> 5k1 pull-up -> 0.1uF decoupler -> BUFFER (3v3 Vcc) -> mcu
5. [x] WLED data outputs (from 3v3 MCU to 5V WS281x LED strips)
    - MCU -> 33R -> BUFFER -> 5k1 pull-down -> 33R -> output node
    - output node -> PMEG -> 5V
    - GND -> TVS -> output node
    - output node -> D+ of twisted pair signal to LED strip
    - D- of twisted pair to GND
6. [x] TPS2HCS08-Q1 ground network:
    - HSS-1 -> PMEG || 5k1 -> GND
7. [x] TPS2HCS10A, Vout2: current design is:
    - HSS-Vout2 -> (A)PMEG-1(K) -> node1 -> Relay-85
    - "dirty GND" -> (A)PMEG-2(K) -> node1
    - Relay-86 -> "dirty GND"
8. [x] Wired-AND - this is a board-wide 3v3 SPI to 5V SPI circuit, that enables or Hi-Z's the shared MISO line to the mcu. if either the TCAN's or the HSS's $\overline{CS}$ is pulled low, then the I/O hat's MISO line is enabled, otherwise it is Hi-Z'd
    - mcu GPIOx -> nodeA -> BUFFER(a) -> ~CAN_CS
    - MCU GPIOy -> nodeB -> BUFFER(b) -> ~HSS_CS
    - common MISO -> BUFFER(c) -> 33R -> MCU GPIOz
    - nodeC -> (A)PMEG-1(K) -> nodeA
    - nodeC -> (A)PMEG-2(K) -> nodeB
    - nodeC -> 5k1 -> 3V3
    - nodeC -> BUFFER(c)-$\overline{OE}$
9. [ ] 2A output support for HSS's $V_{OUT1}$ to the 60 LED SK6812 strip (cannot solve with universal diode.  will need a P-FET:  )
10. [ ] rework the active-high vehicle inputs to remove the voltage dividers and make the inputs universally accept 3V3 - 16V signals.  (requires 2 PMEGS to clamp the voltage into the BUFFER chip)

---
### References
1. TVS = [D15V0H1U2LP-7B](https://www.diodes.com/assets/Datasheets/D15V0H1U2LP.pdf)
1. TPS2121 = Power Mux = [TPS2121](https://www.ti.com/lit/ds/symlink/tps2121.pdf)
1. TCAN = [PCAN4550-Q1](https://www.ti.com/lit/ds/symlink/tcan4550-q1.pdf)
1. PMEG = [PMEG4002ESF](https://assets.nexperia.com/documents/data-sheet/PMEG4002ESF.pdf)
1. HSS = [TPS2HCS10A-Q1](https://www.ti.com/lit/ds/symlink/tps2hcs10-q1.pdf)
1. BUFFER = [74LV1T125](https://assets.nexperia.com/documents/data-sheet/74LV1T125.pdf)
1. Relay - Bosch-style relay



