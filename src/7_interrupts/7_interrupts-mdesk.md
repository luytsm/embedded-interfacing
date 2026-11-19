# Hoofdstuk 7: Interrupts

In de vorige hoofdstukken controleerde het programma zelf, telkens opnieuw, of
er iets veranderd was: `digitalRead()` in [4.4](../4_gpio/4.4_digitale_input.md) leest bij elke doorloop van
`loop()` de toestand van een knop, `Serial.available()` in [5.5](../5_seriele_communicatie/5.5_serial_in_de_arduino_core.md) kijkt bij
elke doorloop of er een byte binnen is. Dat heet **polling**. Polling werkt
goed zolang `loop()` snel genoeg rondgaat. Duurt één doorloop te lang, dan
gaat er informatie verloren. De rotary encoder uit [4.4.7](../4_gpio/4.4_digitale_input.md#447-use-cases) was daar al
een voorbeeld van: wordt er snel gedraaid, dan mist de sketch stappen.

Een **interrupt** pakt dat anders aan. De hardware van de microcontroller
houdt zelf een pin of een peripheral in het oog, en onderbreekt het programma
op het moment dat er iets gebeurt, ongeacht welke regel code op dat moment
uitgevoerd wordt. Het programma hoeft dus niet meer zelf te blijven vragen of
er iets veranderd is.

De volgende secties overlopen:

- **[7.1 De superloop](./7.1_de_superloop.md)**: hoe elke sketch tot nu toe opgebouwd was, de
  voor- en nadelen daarvan, en een korte kadering van een RTOS.
- **[7.2 Wat is een interrupt](./7.2_wat_is_een_interrupt.md)**: wat er gebeurt wanneer een interrupt
  optreedt, wat een ISR is, en polling tegenover interrupts.
- **[7.3 GPIO-interrupts in de Arduino core](./7.3_gpio_interrupts_in_de_arduino_core.md)**: `attachInterrupt()`, welke pinnen
  een interrupt kunnen geven, de interrupt modes, een ISR declareren en een
  interrupt terug verwijderen.
- **[7.4 Gedeelde variabelen en volatile](./7.4_gedeelde_variabelen_en_volatile.md)**: waarom een variabele die de ISR en
  `loop()` delen, `volatile` moet zijn, en waarom dat niet altijd volstaat.
- **[7.5 De ideale ISR](./7.5_de_ideale_isr.md)**: waarom een ISR zo kort mogelijk moet zijn, en wat
  er niet in een ISR thuishoort.
- **[7.6 Vlaggen](./7.6_vlaggen.md)**: het werk verdelen tussen de ISR en `loop()`,
  `attachInterruptArg()`, en *Onder de motorkap*, waarin hetzelfde in het
  ESP-IDF gebeurt.
- **[7.7 Use cases](./7.7_use_cases.md)**: een drukknop debouncen, een rotary
  encoder uitlezen en een motor stoppen met een noodstop.

Dit hoofdstuk focust op interrupts op een **GPIO**: een knop, een sensor of
een encoder aan een pin. Ook andere peripherals, zoals de UART en de timers,
werken met interrupts. Het principe is telkens hetzelfde.
