# Hoofdstuk 1: Introductie tot Embedded Interfacing

## 1.1 Het doel van Embedded Interfacing

Het vak "Embedded Interfacing" draait, zoals de naam laat vermoeden, niet om
één specifiek merk of platform, maar om **embedded interfacing** als proces:
het ontwerpen en programmeren van systemen die diverse componenten
(sensoren, actuatoren, andere elektronische bouwstenen) aansturen met behulp
van een microcontroller — en die, dankzij diezelfde microcontroller, vaak
ook van op afstand bestuurbaar of opvolgbaar zijn. Dat proces staat centraal
doorheen dit vak, en is wat uiteindelijk beheerst moet worden.

Om dat proces aan te leren, vertrekken we van **Arduino**, een platform dat
bij de meeste studenten wellicht al enigszins bekend is, al was het maar
oppervlakkig. Arduino is in essentie een **eenvoudig ecosysteem** waarmee,
met een beperkte leercurve, al snel een werkende embedded applicatie
gebouwd kan worden. Die eenvoud brengt meteen ook een **trade-off** met zich
mee: tegenover de snelheid en toegankelijkheid bij het bouwen van een
eenvoudige applicatie staat een verlies aan performantie.

Als student Elektronica-ICT is het echter de bedoeling verder te groeien dan
die eenvoud, en een meer professionele werkwijze eigen te maken. We starten
daarom bewust met Arduino en zijn toebehoren (de Arduino core, de Arduino
IDE, ...), maar het uiteindelijke doel van dit vak is om tegen het einde
ervan te kunnen werken met het **ecosysteem** ontwikkeld door een
microcontrollerfabrikant zelf. Voor dit vak kiezen we voor het ESP-ecosysteem van Espressif. Het aanleren van het ontwerpen en programmeren van embedded systemen kan zeker ook met andere opties dan ESP: in het werkveld komen evengoed STM32 (STMicroelectronics)
of Nordic Semiconductor (met hun nRF-reeks) voor, elk met hun eigen native
framework.

In dit vak kiezen we specifiek voor het **ESP-IDF** (Espressif IoT
Development Framework), het native ontwikkelframework voor de ESP32-familie
die we in dit vak gebruiken. Die keuze hangt rechtstreeks samen met de
opleiding: binnen Elektronica-ICT ligt de focus op het **Internet of
Things**, dus op het bouwen van **verbonden toepassingen** — apparaten die
verbinding maken met het internet, of die onderling communiceren, bijvoorbeeld
via Bluetooth. ESP-gebaseerde microcontrollers bieden hiervoor het mooiste en
meest toegankelijke aanbod: WiFi en Bluetooth zitten standaard ingebouwd, en
ook andere IoT-protocollen zoals Zigbee of Thread/Matter worden ondersteund.




