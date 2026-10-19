# Hoofdstuk 6: SPI en I²C

In [hoofdstuk 5](../5_seriele_communicatie/5_seriele_communicatie.md) communiceerde de ESP32 via UART met de computer en met externe
modules, zoals een GPS-module. Een UART-verbinding verbindt telkens exact twee
toestellen. Op een printplaat of in een sensormodule moet een microcontroller
echter met een heel aantal andere chips praten: een sensor, een display, een
flashgeheugen, een externe ADC, ... Voor elk van die chips een eigen UART
voorzien is geen optie. Daarvoor zijn twee andere seriële protocollen
ontstaan: **SPI** en **I²C**.

Beide protocollen kwamen kort aan bod in [5.1](../5_seriele_communicatie/5.1_wat_is_seriele_communicatie.md). Een korte herhaling van de
categorieën uit die sectie:

- **Richting** (zie [5.1.2](../5_seriele_communicatie/5.1_wat_is_seriele_communicatie.md#512-richting-simplex-half-duplex-en-full-duplex)): simplex, half duplex of full duplex.
- **Kloksynchronisatie** (zie [5.1.3](../5_seriele_communicatie/5.1_wat_is_seriele_communicatie.md#513-synchroon-versus-asynchroon)): asynchroon, zonder kloklijn, zoals UART,
  of synchroon, met een aparte kloklijn.

SPI en I²C zijn allebei **synchroon**: een master genereert de klok, en de
andere toestellen volgen. Daar houdt de gelijkenis op:

- **SPI** is snel en eenvoudig, maar vraagt veel pinnen: een aparte
  selectielijn per toestel.
- **I²C** heeft maar twee lijnen nodig voor tientallen toestellen, maar is
  trager en heeft een complexer protocol.

De volgende secties overlopen:

- **[6.1 SPI](./6.1_spi.md)**: de kenmerken, het aansluiten, de (nieuwe) namen van de signalen,
  en hoe SPI met schuifregisters werkt.
- **[6.2 SPI-modes](./6.2_spi_modes.md)**: klokpolariteit en klokfase, en hoe de juiste mode uit een
  datasheet gehaald wordt.
- **[6.3 Meerdere SPI-peripherals](./6.3_meerdere_spi_peripherals.md)**: multidrop en daisy chain, peripherals die
  afwijken van de regel, dual en quad SPI, en de voor- en nadelen van SPI.
- **[6.4 I²C](./6.4_i2c.md)**: de kenmerken, snelheden en adressen, en het aansluiten op de
  FireBeetle 2.
- **[6.5 Open drain en pull-up-weerstanden](./6.5_open_drain_en_pull_ups.md)**: waarom elke I²C-lijn een
  pull-up-weerstand nodig heeft, en hoe die berekend wordt.
- **[6.6 Het I²C-protocol](./6.6_het_i2c_protocol.md)**: een transmissie bit per bit, van de startconditie
  tot de stopconditie, en het lezen en schrijven van een register.

Dit hoofdstuk focust op wat er op de lijnen gebeurt. Dat inzicht is nodig om
een datasheet te lezen, de juiste instellingen te kiezen, en een probleem op te
sporen met een logic analyzer wanneer een sensor niet antwoordt.
