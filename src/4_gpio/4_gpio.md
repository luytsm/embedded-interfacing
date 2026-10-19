# Hoofdstuk 4: GPIO's

De vorige hoofdstukken legden de basis: de hardware, de tooling en de
programmeertaal. Dit hoofdstuk zet de eerste stap naar waar het in embedded
interfacing echt om draait: de microcontroller laten **interageren met de
buitenwereld** via zijn pinnen.

De volgende secties overlopen:

- **[4.1 Wat is een GPIO](./4.1_wat_is_een_gpio.md)**: wat een GPIO is, hoe de pinout van de FireBeetle 2
  gelezen wordt, en waarom een GPIO toch niet zo "general purpose" is.
- **[4.2 Welke pin kan wat?](./4.2_welke_pin_kan_wat.md)**: multiplexing, de GPIO matrix van de ESP32, en
  de pinnen die op de FireBeetle 2 beter met rust gelaten worden.
- **[4.3 Digitale output](./4.3_digitale_output.md)**: een pin configureren en aansturen, de
  werkspanning van een board, stroomlimieten en typische toepassingen.
- **[4.4 Digitale input](./4.4_digitale_input.md)**: spanningsniveaus, pull-up en pull-down,
  contactdender en debouncen, en het uitlezen van encoders en pulsen.
- **[4.5 Analoge input](./4.5_analoge_input.md)**: de ADC, zijn resolutie, referentiespanning en
  nauwkeurigheid, en de beperkingen van de ADC in de ESP32.
- **[4.6 PWM](./4.6_pwm.md)**: duty cycle en frequentie, `analogWrite()` en de
  LEDC-peripheral van de ESP32.

Telkens wordt vertrokken van de Arduino-functies. Waar het nuttig is, toont
een laatste sectie *Onder de motorkap* hoe hetzelfde in het ESP-IDF gebeurt.
