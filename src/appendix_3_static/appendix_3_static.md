# Appendix 3: Het keyword static

Elke variabele heeft twee eigenschappen die in de meeste code onopgemerkt
blijven:

- de **scope**: waar in de code de naam zichtbaar is;
- de **levensduur** (*lifetime*): hoe lang de variabele bestaat, en dus hoe
  lang ze haar waarde bijhoudt.

Bij een gewone variabele liggen die twee vast aan elkaar:

- een **lokale** variabele is enkel zichtbaar in haar functie, en verdwijnt
  zodra de functie afgelopen is;
- een **globale** variabele is overal zichtbaar, en bestaat zolang het
  programma loopt.

Het keyword **`static`** maakt het mogelijk om die twee los te koppelen.
Afhankelijk van de plaats waar `static` staat, verandert het één van beide:

- **in een functie** verandert `static` de **levensduur**: de variabele blijft
  enkel zichtbaar in de functie, maar bestaat het hele programma lang en
  onthoudt haar waarde tussen twee oproepen;
- **buiten een functie** verandert `static` de **zichtbaarheid**: een
  variabele of functie is enkel zichtbaar in het bestand waarin ze staat.

Eén woord voor twee verschillende dingen: dat maakt `static` verwarrend.
Toch komen beide in embedded code voortdurend voor:

- in het ESP-IDF staat `static` voor bijna elke functie en variabele, zoals
  `static const char *TAG` (zie [2.3](../2_arduino/2.3_de_arduino_core.md#nog-een-verschil-logging));
- in een sketch lost een `static` variabele in een functie een probleem op dat
  in deze cursus al een paar keer opdook: waarden die bewaard moeten blijven
  tussen twee doorlopen van `loop()` (zie [A1.3.1](../appendix_1_codestandaarden/appendix_1_codestandaarden.md#a131-een-vaste-volgorde) en
  [5.5](../5_seriele_communicatie/5.5_serial_in_de_arduino_core.md)).

Het voorbeeld dat doorheen deze appendix terugkomt, is een **non-blocking
timer**: een led laten knipperen zonder `delay()`.

Deze appendix overloopt:

- **[A3.1 Scope en levensduur](#a31-scope-en-levensduur)**: lokale en globale variabelen, en wat
  `static` daaraan verandert.
- **[A3.2 static in een functie](#a32-static-in-een-functie)**: de regels voor een `static` lokale
  variabele.
- **[A3.3 Use case: een non-blocking timer](#a33-use-case-een-non-blocking-timer)**: van globale variabelen naar
  `static`, en de valkuil van een herbruikbare functie.
- **[A3.4 static buiten een functie](#a34-static-buiten-een-functie)**: zichtbaarheid binnen één bestand, en
  beide betekenissen samen in het ESP-IDF.
- **[A3.5 Waar staat een static variabele in het geheugen?](#a35-waar-staat-een-static-variabele-in-het-geheugen)**: stack tegenover
  vaste plaats in het RAM, en gedeelde toestand.
- **[A3.6 Een tweede voorbeeld: een moving average](#a36-een-tweede-voorbeeld-een-moving-average)**: een ruisfilter voor
  de ADC.
- **[A3.7 Overzicht en keuzehulp](#a37-overzicht-en-keuzehulp)**
- **[A3.8 Oefeningen](#a38-oefeningen)** en **[A3.9 Checklist](#a39-checklist)**

## A3.1 Scope en levensduur

### A3.1.1 Lokale variabelen

Een variabele die binnen een functie gedeclareerd wordt, is **lokaal**:

```cpp
const int SENSOR_PIN = 36;         // A0
const int ALARM_THRESHOLD = 3000;

// Returns true when the sensor value is above the alarm threshold.
bool is_alarm_active() {
  int sensor_value = analogRead(SENSOR_PIN);  // local: only exists during this call
  return sensor_value > ALARM_THRESHOLD;
}
```

- **Scope**: `sensor_value` is enkel zichtbaar binnen de functie, preciezer
  binnen het blok `{ }` waarin ze gedeclareerd is.
  - Een andere functie kan `sensor_value` niet gebruiken.
  - Een andere functie mag wel een eigen variabele met dezelfde naam hebben.
    Dat zijn dan twee aparte variabelen.
- **Levensduur**: de variabele wordt bij elke oproep opnieuw aangemaakt, en
  verdwijnt wanneer de functie afgelopen is. Bij de volgende oproep is er een
  nieuwe variabele, zonder herinnering aan de vorige.

Lokale variabelen staan op de **stack**: een stuk RAM dat bij elke
functie-oproep groeit en bij elke return weer krimpt (zie [A3.5](#a35-waar-staat-een-static-variabele-in-het-geheugen)).

> **Let op:** een lokale variabele zonder beginwaarde heeft een **onbepaalde**
> waarde: wat er toevallig nog op die plaats in de stack stond. Geef een
> lokale variabele dus altijd een beginwaarde voor ze gelezen wordt.

### A3.1.2 Globale variabelen

Een variabele die buiten elke functie gedeclareerd wordt, is **globaal**:

- **Scope**: vanaf de declaratie tot het einde van het bestand, in elke
  functie. Via `extern` kan ook een ander bestand ze gebruiken (zie
  [A3.4](#a34-static-buiten-een-functie)).
- **Levensduur**: het hele programma. De variabele krijgt één keer haar
  beginwaarde, bij het opstarten. Zonder beginwaarde is dat 0.

Daarom gebruikt het voorbeeld `blink_without_delay` in [A1.3.1](../appendix_1_codestandaarden/appendix_1_codestandaarden.md#a131-een-vaste-volgorde) globale
variabelen voor `last_toggle_ms` en `led_on`: die moeten een doorloop van
`loop()` overleven.

Globale variabelen hebben wel nadelen (zie ook [3.4.4](../3_c_en_cpp/3.4_functies.md#344-goede-gewoontes)):

- **Elke functie kan ze aanpassen.** Loopt er iets mis, dan is elke functie
  verdacht.
- **De namen stapelen zich op.** Met drie knipperende leds worden het
  `last_toggle_status_ms`, `last_toggle_alarm_ms`, `last_toggle_error_ms`, en
  zo verder.
- **Het verband is niet zichtbaar.** Wie een functie leest, moet bovenaan de
  sketch gaan zoeken welke globale variabelen ze gebruikt, en welke andere
  functies diezelfde variabelen nog aanpassen.

### A3.1.3 Wat static verandert

|                             | Scope                                          | Levensduur          |
| --------------------------- | ---------------------------------------------- | ------------------- |
| lokale variabele            | de functie                                     | één oproep          |
| globale variabele           | het bestand, en via `extern` andere bestanden  | het hele programma  |
| `static` in een functie     | de functie                                     | het hele programma  |
| `static` buiten een functie | het bestand                                    | het hele programma  |

Een `static` variabele in een functie combineert dus het beste van beide: de
levensduur van een globale variabele, met de scope van een lokale.

## A3.2 static in een functie

### A3.2.1 De regels

Het keyword `static` staat voor het type:

```cpp
void update_blink_led() {
  static unsigned long last_toggle_ms = 0;
  // ...
}
```

Voor zo'n variabele gelden drie regels:

1. **Ze krijgt één keer haar beginwaarde.** De `= 0` wordt niet bij elke
   oproep uitgevoerd, maar één keer. Bij de tweede oproep wordt de regel
   `static unsigned long last_toggle_ms = 0;` dus overgeslagen.
2. **Ze behoudt haar waarde.** Na de return blijft de variabele bestaan. De
   volgende oproep vindt de waarde terug die de vorige oproep achterliet.
3. **Ze blijft enkel zichtbaar in de functie.** Net als een gewone lokale
   variabele kan geen enkele andere functie erbij.

Zonder beginwaarde krijgt een `static` variabele, net als een globale, de
waarde 0. Toch wordt de beginwaarde altijd expliciet geschreven: `= 0` maakt
duidelijk dat 0 een bewuste keuze is.

### A3.2.2 Een constante beginwaarde

> **Let op:** in C en C++ gelden hier andere regels.
>
> - In **C**, en dus in het ESP-IDF, moet de beginwaarde al bij het
>   compileren vastliggen. `static int64_t last_toggle_us = 0;` is in orde,
>   `static int64_t last_toggle_us = esp_timer_get_time();` geeft de
>   foutmelding `initializer element is not constant`.
> - In **C++**, en dus in een sketch, mag de beginwaarde wel het resultaat
>   van een functie zijn. Die wordt dan uitgevoerd bij de eerste oproep.
>   `static unsigned long start_ms = millis();` werkt in een sketch.
>
> Om de code uitwisselbaar te houden tussen een sketch en het ESP-IDF, krijgt
> een `static` variabele in deze cursus altijd een constante beginwaarde.

## A3.3 Use case: een non-blocking timer

### A3.3.1 Het vertrekpunt

Een lange `delay()` houdt de rest van het programma tegen (zie
[Lussen in embedded code](../3_c_en_cpp/3.3_controlestructuren.md#lussen-in-embedded-code)). De non-blocking aanpak met `millis()` kwam al aan bod
bij het debouncen van een drukknop ([4.4.6](../4_gpio/4.4_digitale_input.md#debouncen-in-software)) en bij het inlezen van seriële
data ([5.5](../5_seriele_communicatie/5.5_serial_in_de_arduino_core.md)):

- onthoud het tijdstip van de laatste actie;
- kijk bij elke doorloop van `loop()` of er sindsdien genoeg tijd verstreken
  is;
- zo niet, ga meteen verder met de rest van het programma.

In [A1.3.1](../appendix_1_codestandaarden/appendix_1_codestandaarden.md#a131-een-vaste-volgorde) staan `last_toggle_ms` en `led_on` daarvoor als globale
variabelen bovenaan de sketch. Enkel `loop()` gebruikt ze. Het doel van deze
use case: die twee variabelen verhuizen naar de enige functie die ze nodig
heeft.

### A3.3.2 Een eerste poging: gewone lokale variabelen

De logica van het knipperen verhuist naar een eigen functie,
`update_blink_led()`, met de variabelen erin:

```cpp
// not like this: last_toggle_ms and led_on are reset on every call
void update_blink_led() {
  unsigned long last_toggle_ms = 0;
  bool led_on = false;

  if (millis() - last_toggle_ms >= BLINK_HALF_PERIOD_MS) {
    last_toggle_ms = millis();
    led_on = !led_on;
    digitalWrite(LED_PIN, led_on);
  }
}
```

De compiler geeft geen foutmelding: dit is geldige code. Ze doet alleen niet
wat bedoeld is:

- `last_toggle_ms` begint bij elke oproep opnieuw op 0. Vanaf 500 ms na het
  opstarten is `millis() - 0` altijd groter dan 500, en is de voorwaarde bij
  elke oproep waar.
- `led_on` begint bij elke oproep opnieuw op `false`, en wordt dus telkens
  `true`.

Het resultaat: na een halve seconde gaat de led aan, en blijft ze aan.

### A3.3.3 De oplossing: static

Met `static` voor beide variabelen werkt het wel:

```cpp
const int LED_PIN = 17;                          // D10
const unsigned long BLINK_HALF_PERIOD_MS = 500;  // 1 Hz

// Toggles the LED every BLINK_HALF_PERIOD_MS, without blocking.
void update_blink_led() {
  static unsigned long last_toggle_ms = 0;  // keeps its value between calls
  static bool led_on = false;

  if (millis() - last_toggle_ms >= BLINK_HALF_PERIOD_MS) {
    last_toggle_ms = millis();
    led_on = !led_on;
    digitalWrite(LED_PIN, led_on);
  }
}

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  update_blink_led();
}
```

Wat opvalt:

- Er zijn **geen globale variabelen** meer. Alles wat met de knipperende led
  te maken heeft, staat in één functie. Wie wil weten hoe de led knippert,
  moet maar op één plaats kijken.
- Geen enkele andere functie kan `last_toggle_ms` of `led_on` per ongeluk
  aanpassen.
- `loop()` leest als een inhoudstafel (zie [A1.3.2](../appendix_1_codestandaarden/appendix_1_codestandaarden.md#a132-een-korte-loop)). Een tweede taak
  toevoegen is één extra regel.

### A3.3.4 Rekenen met de tijd rond de overloop

`millis()` geeft een `unsigned long` terug: op de ESP32 is dat 4 bytes, met
als hoogste waarde 4 294 967 295 ms. Na ongeveer **49,7 dagen** loopt de
teller over en begint ze opnieuw bij 0 (zie [3.2.4](../3_c_en_cpp/3.2_datatypes.md#324-let-op-overloop)). Voor een toestel dat
maanden blijft draaien, is dat geen theoretisch probleem.

De vorm `millis() - last_toggle_ms >= BLINK_HALF_PERIOD_MS` werkt ook rond die
overloop. Ook een aftrekking van `unsigned` getallen loopt over, en dat geeft
net het juiste verschil:

- `last_toggle_ms` = 4 294 967 000, net voor de overloop;
- 500 ms later is de teller overgelopen, en geeft `millis()` 204 terug;
- `204 - 4 294 967 000` geeft in `unsigned long` **500**.

Een andere schrijfwijze die op het eerste zicht hetzelfde doet, werkt rond de
overloop **niet**:

```cpp
// not like this: last_toggle_ms + BLINK_HALF_PERIOD_MS overflows
if (millis() >= last_toggle_ms + BLINK_HALF_PERIOD_MS) {
```

Met `last_toggle_ms` = 4 294 967 000 loopt de som over naar 204. Elke waarde
van `millis()` is groter dan 204, en de led wisselt meteen in plaats van na
500 ms.

De regel: tijden zijn altijd `unsigned long`, en er wordt altijd
**afgetrokken**: huidige tijd min vorige tijd, vergeleken met het interval.

### A3.3.5 Valkuil: een gedeelde static

Een tweede led die sneller knippert, vraagt opnieuw dezelfde logica. Het ligt
voor de hand om een algemene hulpfunctie te schrijven:

```cpp
// not like this: one static variable, shared by every caller
bool is_interval_elapsed(unsigned long interval_ms) {
  static unsigned long last_ms = 0;

  if (millis() - last_ms >= interval_ms) {
    last_ms = millis();
    return true;
  }
  return false;
}

void loop() {
  if (is_interval_elapsed(STATUS_BLINK_HALF_PERIOD_MS)) {  // 500 ms
    toggle_status_led();
  }
  if (is_interval_elapsed(ALARM_BLINK_HALF_PERIOD_MS)) {   // 100 ms
    toggle_alarm_led();
  }
}
```

De alarmled knippert, de statusled nooit. Wat gebeurt er?

- Er bestaat maar **één** `last_ms`, hoe vaak en van waar de functie ook
  opgeroepen wordt.
- Op t = 100 ms:
  - de eerste oproep (500 ms) geeft `false`: 100 − 0 is kleiner dan 500;
  - de tweede oproep (100 ms) geeft `true`, en zet `last_ms` op 100.
- Op t = 200 ms zet de tweede oproep `last_ms` op 200, op t = 300 ms op 300,
  en zo verder.
- `last_ms` wordt dus nooit ouder dan 100 ms, en het interval van 500 ms is
  nooit verstreken.

Een `static` variabele hoort bij de **functie**, niet bij wie de functie
oproept. Een functie met een `static` variabele is daarom geschikt voor één
welbepaalde taak, zoals "de statusled laten knipperen", maar niet als
herbruikbaar gereedschap.

### A3.3.6 Een herbruikbare timer

Moet een functie voor meerdere taken bruikbaar zijn, dan verhuist de toestand
naar de **oproeper**. Die toestand wordt gebundeld in een `struct`, en via
een pointer meegegeven (zie [3.4.3](../3_c_en_cpp/3.4_functies.md#343-pass-by-value-en-pass-by-reference)):

```cpp
const int STATUS_LED_PIN = 2;                           // D9, built-in LED
const int ALARM_LED_PIN = 17;                           // D10
const unsigned long STATUS_BLINK_HALF_PERIOD_MS = 500;  // 1 Hz
const unsigned long ALARM_BLINK_HALF_PERIOD_MS = 100;   // 5 Hz

struct interval_timer_t {
  unsigned long interval_ms;
  unsigned long last_ms;
};

// Returns true once every interval_ms. The state lives in the timer, not in this function.
bool is_timer_elapsed(interval_timer_t *timer) {
  if (millis() - timer->last_ms >= timer->interval_ms) {
    timer->last_ms = millis();
    return true;
  }
  return false;
}

void update_status_led() {
  static interval_timer_t blink_timer = { STATUS_BLINK_HALF_PERIOD_MS, 0 };
  static bool led_on = false;

  if (is_timer_elapsed(&blink_timer)) {
    led_on = !led_on;
    digitalWrite(STATUS_LED_PIN, led_on);
  }
}

void update_alarm_led() {
  static interval_timer_t blink_timer = { ALARM_BLINK_HALF_PERIOD_MS, 0 };
  static bool led_on = false;

  if (is_timer_elapsed(&blink_timer)) {
    led_on = !led_on;
    digitalWrite(ALARM_LED_PIN, led_on);
  }
}

void setup() {
  pinMode(STATUS_LED_PIN, OUTPUT);
  pinMode(ALARM_LED_PIN, OUTPUT);
}

void loop() {
  update_status_led();
  update_alarm_led();
}
```

Wat opvalt:

- `is_timer_elapsed()` heeft **geen** `static` variabele meer. De functie
  krijgt de toestand via de pointer `timer`, en is daardoor voor elke timer
  bruikbaar.
- `timer->last_ms` is een kortere schrijfwijze voor `(*timer).last_ms`: het
  veld `last_ms` van de `struct` waar `timer` naar wijst.
- Beide `update_...()`-functies hebben een eigen `blink_timer` en een eigen
  `led_on`. Dezelfde naam in twee functies geeft twee aparte variabelen, die
  elkaar niet storen. Met globale variabelen kan dat niet.
- Het type `interval_timer_t` staat voor de functies, zoals de vaste volgorde
  uit [A1.3.1](../appendix_1_codestandaarden/appendix_1_codestandaarden.md#a131-een-vaste-volgorde) vraagt.

## A3.4 static buiten een functie

### A3.4.1 Zichtbaar in één bestand

In een sketch met één `.ino`-bestand verandert `static` buiten een functie
praktisch niets. In een project met meerdere bestanden, zoals elk
ESP-IDF-project, wel. Een functie die in een `.c`-bestand staat, kan vanuit
een ander bestand opgeroepen worden zodra het prototype gekend is,
bijvoorbeeld via een headerbestand (zie [3.4.2](../3_c_en_cpp/3.4_functies.md#342-declaratie-en-definitie)). Voor een globale
variabele gebeurt hetzelfde met `extern`. Het is de **linker** die bij het
bouwen al die namen uit alle bestanden aan elkaar koppelt.

Dat heeft een gevolg: een globale naam moet **uniek** zijn in het hele
project. Stel een project met drie bestanden:

- `main.c`
- `sensor.c`, met `const char *TAG = "sensor";`
- `display.c`, met `const char *TAG = "display";`

Beide `.c`-bestanden compileren zonder problemen. Pas de linker vindt twee
keer de naam `TAG`, en stopt met een foutmelding als `multiple definition of
'TAG'`. Hetzelfde gebeurt met twee hulpfuncties die toevallig dezelfde naam
hebben, zoals `init_gpio()`.

Met `static` blijft een naam **binnen het bestand**. De linker krijgt hem
niet te zien, en elk bestand heeft zijn eigen `TAG`:

```c
// sensor.c
static const char *TAG = "sensor";

// display.c
static const char *TAG = "display";
```

Zo verdeelt een bestand zijn namen in twee groepen:

- **publiek**: wat in het headerbestand staat, en door andere bestanden
  gebruikt mag worden;
- **privé**: al de rest, met `static` ervoor.

De style guide van het ESP-IDF maakt daar een regel van: *"Any variable or
function which is only used in a single source file should be declared
static."* Daarom staat `static` in het ESP-IDF voor bijna elke hulpfunctie,
zoals `static void set_relay()` in [Appendix 2](../appendix_2_mosfet_schakelen/appendix_2_mosfet_schakelen.md#a245-code).

> **Ter info:** dezelfde style guide vraagt om `static` variabelen het
> voorvoegsel `s_` te geven, zoals `static bool s_invert`. Dat komt veel voor
> in de broncode van het ESP-IDF zelf. Appendix 1 legt dat voorvoegsel niet op.

### A3.4.2 Beide betekenissen samen: de timer in het ESP-IDF

In het ESP-IDF bestaan `setup()` en `loop()` niet (zie
[4.3.6](../4_gpio/4.3_digitale_output.md#436-onder-de-motorkap-het-esp-idf)). `app_main()` wordt één keer uitgevoerd, en de eindeloze lus
wordt er zelf in geschreven. De non-blocking timer ziet er dan zo uit:

```c
#include <stdbool.h>
#include "driver/gpio.h"
#include "esp_timer.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

#define LED_GPIO GPIO_NUM_17          // D10
#define BLINK_HALF_PERIOD_US 500000   // 1 Hz
#define LOOP_PERIOD_MS 10

// Toggles the LED every BLINK_HALF_PERIOD_US, without blocking.
static void update_blink_led(void) {
  static int64_t last_toggle_us = 0;  // keeps its value between calls
  static bool led_on = false;

  int64_t now_us = esp_timer_get_time();  // time since startup in µs
  if (now_us - last_toggle_us >= BLINK_HALF_PERIOD_US) {
    last_toggle_us = now_us;
    led_on = !led_on;
    gpio_set_level(LED_GPIO, led_on);
  }
}

void app_main(void) {
  gpio_config_t led_config = {
    .pin_bit_mask = 1ULL << LED_GPIO,
    .mode = GPIO_MODE_OUTPUT,
    .pull_up_en = GPIO_PULLUP_DISABLE,
    .pull_down_en = GPIO_PULLDOWN_DISABLE,
    .intr_type = GPIO_INTR_DISABLE,
  };
  ESP_ERROR_CHECK(gpio_config(&led_config));

  while (1) {
    update_blink_led();
    vTaskDelay(pdMS_TO_TICKS(LOOP_PERIOD_MS));  // let the other tasks run
  }
}
```

In dit fragment staat `static` twee keer, met telkens een andere betekenis:

- `static void update_blink_led(void)`: **buiten** een functie. De functie is
  enkel zichtbaar in dit bestand.
- `static int64_t last_toggle_us` en `static bool led_on`: **in** een functie.
  De variabelen onthouden hun waarde tussen twee oproepen.

Nog enkele verschillen met de sketch:

- **`esp_timer_get_time()`** geeft de tijd sinds het opstarten in
  **microseconden**, als `int64_t`. Dat is een type met vaste grootte (zie
  [3.2.2](../3_c_en_cpp/3.2_datatypes.md#322-types-met-vaste-grootte)) dat de API oplegt. Met 64 bits loopt die teller pas na
  ongeveer 292 000 jaar over.
- **Onder de motorkap** gebruikt ook `millis()` in de ESP32-core deze functie:
  `millis()` deelt de waarde van `esp_timer_get_time()` door 1000, en geeft het
  resultaat terug als `unsigned long` van 32 bits. Daar komt de overloop na
  49,7 dagen uit [A3.3.4](#a334-rekenen-met-de-tijd-rond-de-overloop) vandaan.
- **`vTaskDelay()`** in de lus is nodig. `app_main()` draait in een
  FreeRTOS-taak. Een lus die nooit wacht, houdt de processor bezet, en dan
  grijpt na enkele seconden de *task watchdog* in. Met een wachttijd van
  10 ms knippert de led tot op 10 ms nauwkeurig, ruim voldoende voor een led.

## A3.5 Waar staat een static variabele in het geheugen?

> **Per traject:**
>
> - **IT & IMT**: te lezen.
> - **IT & IoT**: te kennen.

### A3.5.1 Stack en vaste plaatsen in het RAM

Variabelen komen op verschillende plaatsen in het RAM terecht:

| Plaats              | Wat staat er                                         | Wanneer gereserveerd                         |
| ------------------- | ---------------------------------------------------- | -------------------------------------------- |
| **stack**           | lokale variabelen en parameters                      | bij elke oproep, en weer vrij bij de return  |
| **`.data`**         | globale en `static` variabelen met een beginwaarde   | vast, al bij het bouwen                      |
| **`.bss`**          | globale en `static` variabelen met beginwaarde 0     | vast, al bij het bouwen                      |
| **heap**            | geheugen dat tijdens het uitvoeren aangevraagd wordt, zoals een `String` (zie [5.7.1](../5_seriele_communicatie/5.7_tekst_zonder_string.md#571-reflectie-waarom-geen-string)) | tijdens het uitvoeren |

Een `static` variabele in een functie staat dus **niet** op de stack, maar op
een vaste plaats, net als een globale variabele. Enkel de compiler beperkt
wie erbij mag.

Bij het opstarten worden de beginwaarden van `.data` uit het flashgeheugen
naar het RAM gekopieerd, en wordt `.bss` op 0 gezet. Daarom krijgt een
`static` variabele haar beginwaarde maar één keer: die toekenning gebeurt
niet in de functie, maar al bij het opstarten.

### A3.5.2 Grote buffers

Op de ESP32 heeft elke FreeRTOS-taak haar eigen, vaste stack:

- `loop()` draait in de taak `loopTask`, met standaard 8 KB stack;
- `app_main()` draait in de taak `main`, met standaard 3584 bytes stack.

Een lokale buffer zoals `char log_buffer[4096]` past dus niet op de stack van
`app_main()`. Het programma compileert zonder waarschuwing, maar loopt tijdens
het uitvoeren over het einde van de stack. FreeRTOS merkt dat meestal op, meldt
een *stack overflow* en herstart de ESP32.

Met `static` verhuist de buffer naar `.bss`:

- de buffer neemt geen plaats meer in op de stack;
- de grootte telt mee in het overzicht na het compileren. In de Arduino IDE is
  dat de regel *Global variables use … bytes of dynamic memory*. Is het RAM te
  klein, dan blijkt dat al bij het bouwen, en niet pas tijdens het uitvoeren.

Daar staat één nadeel tegenover: een `static` buffer neemt **altijd** RAM in,
ook wanneer de functie niet uitgevoerd wordt. Een lokale buffer neemt enkel
ruimte in tijdens de oproep. Met de ruim 300 KB RAM van de ESP32 is dat zelden
een probleem, met de 2 KB RAM van een Arduino Nano wel.

### A3.5.3 Gedeelde toestand

Een `static` variabele bestaat maar één keer (zie [A3.3.5](#a335-valkuil-een-gedeelde-static)). Wordt de
functie op twee plaatsen tegelijk uitgevoerd, dan delen beide uitvoeringen
dezelfde variabele en zitten ze elkaar in de weg. Dat gebeurt:

- wanneer twee FreeRTOS-taken dezelfde functie oproepen;
- wanneer een functie zowel vanuit `loop()` als vanuit een interrupt
  opgeroepen wordt.

Een bekend voorbeeld uit de standaardbibliotheek van C is `strtok()`, dat een
tekst opsplitst in stukken. `strtok()` onthoudt in een interne `static`
variabele waar het gebleven is. Twee teksten door elkaar opsplitsen lukt
daardoor niet. Daarom bestaat er ook `strtok_r()`, waarbij de oproeper die
toestand zelf bijhoudt, precies zoals de `interval_timer_t` uit [A3.3.6](#a336-een-herbruikbare-timer).

### A3.5.4 Niet te verwarren: static, const en volatile

Drie keywords die vaak samen voorkomen, maar elk iets anders doen:

- **`const`**: de waarde kan niet meer veranderen (zie [A1.2.2](../appendix_1_codestandaarden/appendix_1_codestandaarden.md#a122-const-in-plaats-van-define)).
- **`static`**: in een functie de levensduur, buiten een functie de
  zichtbaarheid.
- **`volatile`**: de compiler moet de waarde telkens opnieuw uit het geheugen
  lezen, omdat iets buiten de normale programmaflow ze kan veranderen, zoals
  een interrupt (zie [7.4](../7_interrupts/7.4_gedeelde_variabelen_en_volatile.md)).

Ze zijn te combineren. `static const char *TAG` is een tekst die nooit
verandert (`const`), en die enkel in dit bestand zichtbaar is (`static`).

## A3.6 Een tweede voorbeeld: een moving average

### A3.6.1 Het idee

Een meting met de ADC is nooit helemaal stabiel: ook bij een potentiometer
die niet beweegt, verspringt de waarde telkens een beetje (zie
[4.5](../4_gpio/4.5_analoge_input.md)). Een eenvoudig filter daartegen is het **gemiddelde van de
laatste N metingen**, een *moving average*:

- bij elke nieuwe meting valt de oudste meting weg;
- het gemiddelde schuift zo mee met de metingen.

Daarvoor moet de functie de laatste N metingen onthouden tussen twee
oproepen. Ook dat is een taak voor `static`.

### A3.6.2 De code

De metingen staan in een array die gebruikt wordt als **circulaire buffer**:
een index wijst naar de plaats van de oudste meting, die door de nieuwe
overschreven wordt. Na de laatste plaats springt de index terug naar het
begin. De som wordt bijgehouden in plaats van telkens opnieuw berekend.

De timer uit [A3.3.6](#a336-een-herbruikbare-timer) zorgt voor een vast tempo: elke 10 ms een meting,
elke 500 ms een waarde naar de Serial Monitor.

```cpp
const int POT_PIN = 36;                     // A0
const int FILTER_SIZE = 8;                  // number of readings in the average
const unsigned long SAMPLE_PERIOD_MS = 10;  // 100 readings per second
const unsigned long PRINT_PERIOD_MS = 500;

struct interval_timer_t {
  unsigned long interval_ms;
  unsigned long last_ms;
};

// Returns true once every interval_ms. The state lives in the timer, not in this function.
bool is_timer_elapsed(interval_timer_t *timer) {
  if (millis() - timer->last_ms >= timer->interval_ms) {
    timer->last_ms = millis();
    return true;
  }
  return false;
}

// Reads the potentiometer and returns the average of the last FILTER_SIZE readings.
int read_pot_filtered() {
  static int readings[FILTER_SIZE] = { 0 };  // the last FILTER_SIZE readings
  static int next_index = 0;                 // position of the oldest reading
  static int sum = 0;                        // sum of all values in readings

  sum -= readings[next_index];  // the oldest reading leaves the average
  readings[next_index] = analogRead(POT_PIN);
  sum += readings[next_index];

  next_index++;
  if (next_index == FILTER_SIZE) {
    next_index = 0;  // back to the start: a circular buffer
  }

  return sum / FILTER_SIZE;
}

void setup() {
  Serial.begin(115200);
}

void loop() {
  static interval_timer_t sample_timer = { SAMPLE_PERIOD_MS, 0 };
  static interval_timer_t print_timer = { PRINT_PERIOD_MS, 0 };
  static int pot_filtered = 0;

  if (is_timer_elapsed(&sample_timer)) {
    pot_filtered = read_pot_filtered();
  }
  if (is_timer_elapsed(&print_timer)) {
    Serial.println(pot_filtered);
  }
}
```

Wat opvalt:

- De drie `static` variabelen van het filter zijn van buitenaf onbereikbaar.
  Geen enkele andere functie kan de buffer of de som per ongeluk verstoren.
- Ook in `loop()` staan `static` variabelen: `loop()` is gewoon een functie
  die steeds opnieuw opgeroepen wordt. `pot_filtered` moet `static` zijn: in
  een doorloop waarin enkel de printtimer verstreken is, moet de laatst
  berekende waarde nog beschikbaar zijn.
- De som bijhouden kost per meting een aftrekking en een optelling, ongeacht
  `FILTER_SIZE`. Telkens alle metingen opnieuw optellen kost er
  `FILTER_SIZE`.
- **Bij het opstarten** staan er nog nullen in de buffer. De eerste acht
  waarden klimmen daardoor geleidelijk van 0 naar de echte waarde: na 80 ms is
  het filter gevuld.
- Dezelfde valkuil als in [A3.3.5](#a335-valkuil-een-gedeelde-static): `read_pot_filtered()` werkt voor **één**
  potentiometer. Voor een tweede sensor verhuist de toestand naar een
  `struct`, net als bij de timer (zie [A3.8](#a38-oefeningen), oefening 5).

## A3.7 Overzicht en keuzehulp

|                  | Lokaal                     | `static` in een functie                       | `static` buiten een functie                    | Globaal                         |
| ---------------- | -------------------------- | --------------------------------------------- | ---------------------------------------------- | ------------------------------- |
| Zichtbaar in     | de functie                 | de functie                                    | het bestand                                    | het hele project                |
| Bestaat          | tijdens één oproep         | het hele programma                            | het hele programma                             | het hele programma              |
| Geheugen         | stack                      | `.data` of `.bss`                             | `.data` of `.bss`                              | `.data` of `.bss`               |
| Beginwaarde      | onbepaald, zelf toekennen  | één keer, bij het opstarten (standaard 0)     | één keer, bij het opstarten (standaard 0)      | één keer, bij het opstarten     |
| Typisch gebruik  | tussenresultaten           | toestand die maar één functie nodig heeft     | toestand of hulpfuncties van één bestand       | zo weinig mogelijk              |

### Welke variabele?

1. **Standaard: lokaal.**
2. **Moet de waarde een oproep overleven, en heeft maar één functie haar
   nodig?** Een `static` variabele in die functie.
3. **Delen meerdere functies van hetzelfde bestand de waarde?**
   - In het ESP-IDF: een `static` variabele buiten de functies.
   - In een sketch: een globale variabele (zie [A1.3.1](../appendix_1_codestandaarden/appendix_1_codestandaarden.md#a131-een-vaste-volgorde)).
4. **Moet een functie herbruikbaar zijn**, voor meerdere leds, sensoren of
   timers? Dan geen `static` in die functie: de toestand komt in een `struct`
   bij de oproeper, en wordt via een pointer meegegeven.
5. **Altijd in het ESP-IDF:** elke functie en variabele die enkel in dit
   bestand gebruikt wordt, krijgt `static`.

### Verder kijken

- **Een derde betekenis in C++.** In een `class` betekent `static` nog iets
  anders: één variabele die gedeeld wordt door alle objecten van die klasse.
  Een klasse is in C++ ook de gebruikelijke manier om toestand en functies te
  bundelen: veel Arduino-libraries hebben een timerklasse in plaats van een
  `struct` met een losse functie.
- **Timers in het ESP-IDF.** Het ESP-IDF kan zelf een functie periodiek
  oproepen, met de callbacks van `esp_timer` of met de *software timers* van
  FreeRTOS. Dan is er geen lus meer nodig die telkens de tijd controleert.
  Dat valt buiten deze appendix.

## A3.8 Oefeningen

1. **Voorspellen.** In `update_blink_led()` uit [A3.3.3](#a333-de-oplossing-static) staat `static` enkel
   nog voor `last_toggle_ms`, niet meer voor `led_on`. Wat doet de led? Leg uit
   waarom.
2. **Flankdetectie.** Een drukknop hangt aan D2 (GPIO25), met de interne
   pull-up: losgelaten is HIGH, ingedrukt is LOW. Schrijf een functie
   `bool was_button_pressed()` die precies één keer `true` teruggeeft per druk
   op de knop, op het moment dat de knop ingedrukt wordt. Gebruik geen
   globale variabelen.
3. **Seriële regels zonder globale variabelen.** De non-blocking versie in
   [5.5](../5_seriele_communicatie/5.5_serial_in_de_arduino_core.md) gebruikt de globale variabelen `line_buffer` en `line_length`.
   Herschrijf de code zodat het inlezen in een functie `handle_serial_input()`
   staat, zonder globale variabelen. `loop()` roept die functie en
   `update_blink_led()` uit [A3.3.3](#a333-de-oplossing-static) op.
4. **Twee bestanden.** Een ESP-IDF-project bevat `sensor.c` en `display.c`. In
   beide staat bovenaan `const char *TAG = ...;`, zonder `static`. Wat gebeurt
   er bij het bouwen? In welke stap: compileren of linken?
5. **Twee potentiometers.** Pas het filter uit [A3.6](#a36-een-tweede-voorbeeld-een-moving-average) aan zodat het voor
   twee potentiometers werkt, op A0 (GPIO36) en A1 (GPIO39). Gebruik een
   `struct pot_filter_t` en een functie
   `int read_filtered(pot_filter_t *filter)`.
6. **Stack** (IT & IoT). Een functie die vanuit `app_main()` opgeroepen wordt,
   bevat de lokale buffer `char log_buffer[4096]`. Wat gebeurt er? Geef twee
   oplossingen, en het nadeel van elk.

<details>
<summary>Oplossing</summary>

1. De led gaat na 500 ms aan en blijft aan. `last_toggle_ms` werkt wel: de
   voorwaarde is netjes elke 500 ms waar. Maar `led_on` begint bij elke
   oproep opnieuw op `false`, en wordt dus telkens `true`. `digitalWrite()`
   zet de led elke 500 ms opnieuw aan, en nooit uit.

2. De functie onthoudt het vorige niveau, en geeft enkel `true` bij de
   overgang van HIGH naar LOW (een dalende flank):

   ```cpp
   const int BUTTON_PIN = 25;  // D2, button to GND, internal pull-up

   // Returns true once per press: on the change from released (HIGH) to pressed (LOW).
   bool was_button_pressed() {
     static int last_level = HIGH;  // released at startup

     int level = digitalRead(BUTTON_PIN);
     bool is_falling_edge = (last_level == HIGH && level == LOW);
     last_level = level;
     return is_falling_edge;
   }

   void setup() {
     pinMode(BUTTON_PIN, INPUT_PULLUP);
   }
   ```

   Door contactdender kan de functie bij één druk toch meerdere keren `true`
   teruggeven. Debouncen, zoals in [4.4.6](../4_gpio/4.4_digitale_input.md#debouncen-in-software), lost dat op. Ook de
   variabelen van dat voorbeeld kunnen `static` worden.

3. ```cpp
   const int LINE_BUFFER_SIZE = 32;  // longest line + '\0'

   // Collects the characters that have already arrived, and prints a line once it is complete.
   void handle_serial_input() {
     static char line_buffer[LINE_BUFFER_SIZE];
     static int line_length = 0;

     while (Serial.available() > 0) {
       char received_char = Serial.read();

       if (received_char == '\n') {
         line_buffer[line_length] = '\0';  // line complete: close the C-string
         Serial.print("received: ");
         Serial.println(line_buffer);
         line_length = 0;                  // ready for the next line
       } else if (received_char != '\r' && line_length < LINE_BUFFER_SIZE - 1) {
         line_buffer[line_length] = received_char;
         line_length++;
       }
     }
   }

   void loop() {
     handle_serial_input();
     update_blink_led();
   }
   ```

   Een regel kan verspreid over meerdere oproepen binnenkomen. Daarom moeten
   zowel `line_buffer` als `line_length` `static` zijn: anders gaan de tekens
   die al ontvangen zijn, verloren bij de return.

4. Beide bestanden **compileren** zonder fout: elk bestand op zich is in orde.
   Het **linken** mislukt, met een melding als `multiple definition of 'TAG'`.
   De oplossing: `static const char *TAG = ...;` in beide bestanden.

5. ```cpp
   const int POT_A_PIN = 36;  // A0
   const int POT_B_PIN = 39;  // A1
   const int FILTER_SIZE = 8;

   struct pot_filter_t {
     int pin;
     int readings[FILTER_SIZE];
     int next_index;
     int sum;
   };

   // Reads the potentiometer of this filter and returns the average of the last FILTER_SIZE readings.
   int read_filtered(pot_filter_t *filter) {
     filter->sum -= filter->readings[filter->next_index];
     filter->readings[filter->next_index] = analogRead(filter->pin);
     filter->sum += filter->readings[filter->next_index];

     filter->next_index++;
     if (filter->next_index == FILTER_SIZE) {
       filter->next_index = 0;
     }

     return filter->sum / FILTER_SIZE;
   }

   void loop() {
     static pot_filter_t filter_a = { POT_A_PIN, { 0 }, 0, 0 };
     static pot_filter_t filter_b = { POT_B_PIN, { 0 }, 0, 0 };

     int pot_a = read_filtered(&filter_a);
     int pot_b = read_filtered(&filter_b);
     // ...
   }
   ```

   `read_filtered()` heeft geen `static` variabelen meer. De twee filters
   staan als `static` variabelen in `loop()`, zodat ze bewaard blijven tussen
   twee doorlopen.

6. 4096 bytes is meer dan de 3584 bytes stack van de taak `main`. Het
   programma loopt over het einde van de stack: FreeRTOS meldt meestal een
   *stack overflow* en de ESP32 herstart. Als het niet opgemerkt wordt, worden
   andere gegevens overschreven, met onvoorspelbare fouten tot gevolg.
   - **De buffer `static` maken.** Hij verhuist naar `.bss`, en telt mee in het
     geheugenoverzicht bij het bouwen. Nadeel: de 4 KB RAM blijven altijd
     bezet, en de functie kan niet meer veilig vanuit twee taken tegelijk
     opgeroepen worden.
   - **De stack van de taak `main` groter maken**, met de instelling
     `CONFIG_ESP_MAIN_TASK_STACK_SIZE` in `idf.py menuconfig`. Nadeel: dat RAM
     is voor die taak gereserveerd, ook wanneer de functie niet uitgevoerd
     wordt, en de stack moet opnieuw vergroot worden zodra er nog een grote
     buffer bij komt.

</details>

## A3.9 Checklist

- [ ] Moet een waarde een oproep overleven? Dan is ze `static` of globaal, geen
      gewone lokale variabele.
- [ ] Gebruikt maar één functie die waarde? Dan een `static` variabele in die
      functie, geen globale.
- [ ] Heeft elke `static` variabele een expliciete, constante beginwaarde?
- [ ] Een herbruikbare functie zonder `static`: de toestand zit in een `struct`
      bij de oproeper?
- [ ] Wordt een functie met een `static` variabele niet vanuit twee taken, of
      vanuit `loop()` en een interrupt, opgeroepen?
- [ ] Tijden in `unsigned long`, en altijd `millis() - last_ms >= interval_ms`?
- [ ] Een grote buffer: bewust gekozen tussen de stack en `static`?
- [ ] ESP-IDF: elke functie en variabele die enkel in dit bestand gebruikt
      wordt, heeft `static`?

## Bronnen

- [cppreference, Storage-class specifiers (C)](https://en.cppreference.com/w/c/language/storage_class_specifiers)
- [cppreference, Storage class specifiers (C++)](https://en.cppreference.com/w/cpp/language/storage_duration)
- [Arduino, Language Reference: static](https://docs.arduino.cc/language-reference/en/variables/variable-scope-qualifiers/static/)
- [Espressif, ESP-IDF Style Guide](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/contribute/style-guide.html)
- [Espressif, ESP-IDF – ESP Timer (High Resolution Timer)](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/esp_timer.html)
- [Espressif, ESP-IDF – Fatal Errors](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/fatal-errors.html)
