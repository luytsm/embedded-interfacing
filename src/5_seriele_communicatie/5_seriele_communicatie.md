# Hoofdstuk 5: Seriële communicatie

In [hoofdstuk 4](../4_gpio/4_gpio.md) communiceerde de microcontroller met de buitenwereld via
individuele pinnen: één pin, één bit, HIGH of LOW. Een temperatuur, een
GPS-positie of een regel tekst voor de Serial Monitor bestaan echter uit
tientallen of honderden bits. Voor elk bit een eigen pin voorzien is geen
optie. **Seriële communicatie** lost dat op: de bits worden één voor één, na
elkaar, over één draad verstuurd.

Seriële communicatie werd eigenlijk al gebruikt vanaf [hoofdstuk 2](../2_arduino/2_arduino.md). Elke
`Serial.print()` stuurt tekst serieel naar de computer, en ook het uploaden
van een sketch verloopt zo. Dit hoofdstuk kijkt wat er achter die verbinding
zit: hoe de bits over de draad gaan, hoe ze van de ESP32 tot op de computer
geraken, en hoe de seriële poort in een sketch gebruikt wordt zonder het
programma te vertragen.

De volgende secties overlopen:

- **[5.1 Wat is seriële communicatie](./5.1_wat_is_seriele_communicatie.md)**: serieel en parallel, de richting van
  de data, synchroon en asynchroon, en de meest gebruikte protocollen.
- **[5.2 UART](./5.2_uart.md)**: het protocol achter `Serial`: aansluiten, baudrate, de opbouw
  van een frame en hoe snel een UART-verbinding is.
- **[5.3 RS-232, RS-422 en RS-485](./5.3_rs232_rs422_rs485.md)**: hetzelfde frame over een langere afstand,
  met andere spanningsniveaus.
- **[5.4 Van USB naar UART](./5.4_van_usb_naar_uart.md)**: hoe de FireBeetle 2 via USB met de computer
  praat, en welke UART's de ESP32 aan boord heeft.
- **[5.5 Serial in de Arduino core](./5.5_serial_in_de_arduino_core.md)**: de belangrijkste functies, functies die
  wachten, en een tweede UART voor een externe module.
- **[5.6 Tekst en bytes](./5.6_tekst_en_bytes.md)**: ASCII, regeleindes, van teken naar getal, en
  binaire data met endianness.
- **[5.7 Tekst zonder String](./5.7_tekst_zonder_string.md)**: waarom `char`-arrays en geen `String`, en de
  belangrijkste functies uit `string.h` in de praktijk.

Telkens wordt vertrokken van de Arduino-functies. Sectie [5.5](./5.5_serial_in_de_arduino_core.md) sluit af met
*Onder de motorkap*, waarin hetzelfde in het ESP-IDF gebeurt.
