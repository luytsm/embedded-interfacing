# Appendix 1: Codestandaarden

Code wordt veel vaker gelezen dan geschreven. Een sketch die vandaag
geschreven wordt, wordt later opnieuw gelezen: door de auteur zelf een paar
weken later, door een collega die een fout zoekt, of door een docent die ze
beoordeelt. Een **codestandaard** legt afspraken vast over naamgeving, opbouw
en opmaak. Daardoor ziet alle code er hetzelfde uit, en kan de lezer zich
concentreren op wat de code doet in plaats van op hoe ze geschreven is.

Elke grote codebase heeft zo'n standaard. De Linux-kernel heeft de *Linux
kernel coding style*, Google heeft een *C++ Style Guide*, en Espressif heeft
een **style guide voor het ESP-IDF**. De standaarden in dit hoofdstuk volgen
grotendeels die van het ESP-IDF, zodat de overstap van Arduino naar het
ESP-IDF later vanzelf gaat. Enkel de opmaak volgt de Arduino IDE (zie [A1.4](#a14-opmaak-en-commentaar)).

Deze standaarden gelden voor alle code in dit vak: de voorbeelden in de
cursus en de code die ingediend wordt voor de labo's.

Deze appendix overloopt:

- **[A1.1 Naamgeving](#a11-naamgeving)**: hoe variabelen, functies, constanten en types
  geschreven worden, en wat een goede naam is.
- **[A1.2 Constanten](#a12-constanten)**: waarom getallen een naam krijgen, en waarom `const`
  de voorkeur krijgt op `#define`.
- **[A1.3 Opbouw van een sketch](#a13-opbouw-van-een-sketch)**: de vaste volgorde van de onderdelen, en een
  korte `loop()`.
- **[A1.4 Opmaak en commentaar](#a14-opmaak-en-commentaar)**: inspringen, accolades en zinvol commentaar.

## A1.1 Naamgeving

Een naam is de eerste documentatie van een stuk code. Een goede naam zegt wat
een variabele bevat of wat een functie doet, zodat er geen commentaar nodig
is om het uit te leggen. Twee dingen bepalen een naam:

- de **schrijfwijze**: hoe de woorden aan elkaar geschreven worden;
- de **inhoud**: welke woorden er gekozen worden.

### A1.1.1 Schrijfwijze

| Wat                                   | Schrijfwijze                    | Voorbeeld                         |
| ------------------------------------- | ------------------------------- | --------------------------------- |
| variabelen en functies                | `snake_case`                    | `pot_value`, `read_temperature()` |
| constanten en macro's (`#define`)     | `UPPER_SNAKE_CASE`              | `LED_PIN`, `DEBOUNCE_DELAY_MS`    |
| waarden van een `enum`                | `UPPER_SNAKE_CASE`              | `SELECT_LED`, `BLINK`             |
| zelfgemaakte types (`enum`, `struct`) | `snake_case` met `_t` achteraan | `state_t`, `button_t`             |

- **snake_case**: kleine letters, de woorden gescheiden door een underscore.
- **UPPER_SNAKE_CASE**: hoofdletters, ook gescheiden door een underscore. Zo
  is in één oogopslag te zien dat een waarde vastligt.
- Het achtervoegsel **`_t`** (van *type*) is dezelfde conventie als in het
  ESP-IDF: `esp_err_t`, `gpio_num_t`, `gpio_config_t`. Ook de types met vaste
  grootte uit [3.2.2](../3_c_en_cpp/3.2_datatypes.md#322-types-met-vaste-grootte) volgen ze, zoals `uint8_t`.

Alle vier samen in één fragment:

```cpp
enum state_t { SELECT_LED, SELECT_FREQ, BLINK };  // type and its values

const int LED_PIN = 17;                           // constant (D10)
state_t state = SELECT_LED;                       // variable

void update_leds(state_t current_state);          // function
```

`UPPER_SNAKE_CASE` is voorbehouden voor waarden die al bij het compileren
vastliggen, zoals een pin, een grenswaarde of een tijdsduur. Een
`const`-parameter van een functie valt daar niet onder: zijn waarde ligt pas
vast bij de oproep. `const char *text` blijft dus `snake_case`.

Een tekst die nooit verandert, is wel een constante. De tag voor logging uit
[2.3](../2_arduino/2.3_de_arduino_core.md) heet daarom `TAG`, zoals ook in de voorbeelden van het ESP-IDF:

```c
static const char *TAG = "distance_sensor";
```

### A1.1.2 Inhoud van een naam

- Code is in het **Engels**: namen én commentaar. Zo sluit de code aan bij
  de functies uit de core en het ESP-IDF, en bij de documentatie en
  voorbeelden online.
- Kies **beschrijvende** namen:
  - `blink_freq_hz` zegt meer dan `f`, `encoder_a_level` meer dan `a`;
  - enkel een lusteller mag `i` heten;
  - vermijd onnodige afkortingen: `measurement`, niet `meas`.
- Een functie begint met een **werkwoord** en doet één ding (zie [3.4.4](../3_c_en_cpp/3.4_functies.md#344-goede-gewoontes)):
  `read_temperature()`, `set_motor_speed()`, `update_leds()`.
- Een **`bool`** leest als een ja/nee-vraag: `is_pressed`, `is_night_mode`,
  `led_on`. Een `if (is_pressed)` leest dan als een gewone zin.
- Zet de **eenheid** in de naam bij tijden, frequenties en andere grootheden
  waar verwarring mogelijk is: `debounce_delay_ms`, `echo_us`,
  `blink_freq_hz`, `voltage_mv`. Of een waarde in milliseconden of
  microseconden staat, is uit het getal zelf niet af te leiden, en een fout
  daarin scheelt meteen een factor 1000.

## A1.2 Constanten

### A1.2.1 Geen magic numbers

Een **magic number** is een getal in de code waarvan de betekenis niet uit de
code zelf blijkt. Dat geeft twee problemen:

- de lezer moet raden waar het getal voor staat;
- komt hetzelfde getal op meerdere plaatsen voor, dan moet een wijziging
  overal gebeuren, en wordt er makkelijk één plaats vergeten.

De oplossing is eenvoudig: elk getal met een betekenis krijgt een constante
met een naam. Dat geldt in het bijzonder voor pinnummers, grenswaarden en
tijden.

```cpp
// not like this
pinMode(17, OUTPUT);
if (analogRead(36) > 3000) {
  digitalWrite(17, HIGH);
}
```

```cpp
// but like this
const int LED_PIN = 17;                  // D10
const int SENSOR_PIN = 36;               // A0
const int ALARM_THRESHOLD = 3000;

pinMode(LED_PIN, OUTPUT);
if (analogRead(SENSOR_PIN) > ALARM_THRESHOLD) {
  digitalWrite(LED_PIN, HIGH);
}
```

Moet de LED later naar een andere pin, dan verandert er één regel in plaats
van twee. Bovendien leest de `if` nu als een zin.

Uitzonderingen:

- `0` en `1`, bijvoorbeeld als beginwaarde van een teller of als stap in een
  lus;
- de baudrate in `Serial.begin(115200)`: die staat maar op één plaats en is in
  elke sketch dezelfde;
- vaste factoren in een formule, zoals `/ 2` voor een afstand die heen en
  terug afgelegd wordt, of `* 1000` om seconden naar milliseconden om te
  zetten.

### A1.2.2 const in plaats van #define

Een constante kan op twee manieren gemaakt worden:

- met **`#define`**: de preprocessor vervangt de naam overal door de tekst
  erachter, nog voor de compiler de code te zien krijgt. De compiler ziet dus
  enkel een getal, zonder type.
- met **`const`**: een variabele met een type, die niet meer kan veranderen.
  De compiler controleert het type, en waarschuwt bijvoorbeeld wanneer een
  waarde niet in dat type past.

In een sketch wordt daarom `const` gebruikt. In het ESP-IDF komt `#define`
veel voor (zie [4.3.6](../4_gpio/4.3_digitale_output.md#436-onder-de-motorkap-het-esp-idf)). Beide volgen dezelfde naamgeving:

```cpp
#define LED_GPIO GPIO_NUM_17   // ESP-IDF: macro, no type
const int LED_PIN = 17;        // sketch: constant with a type
```

## A1.3 Opbouw van een sketch

### A1.3.1 Een vaste volgorde

Wanneer elke sketch dezelfde volgorde volgt, weet de lezer meteen waar iets
staat: de pinnen en instellingen bovenaan, de logica onderaan. De onderdelen
staan altijd in deze volgorde:

1. een **header** in commentaar: bestandsnaam, wat de sketch doet en de
   auteur;
2. de `#include`s;
3. de **constanten**: pinnen, grenswaarden, tijden;
4. de **types**: `enum`, `struct`;
5. de **globale variabelen**;
6. de **functies**;
7. `setup()` en `loop()`.

Onderdelen die niet nodig zijn, vallen gewoon weg.

De functies staan vóór `setup()` en `loop()`, omdat een functie in C gekend
moet zijn voordat ze gebruikt wordt (zie [3.4.2](../3_c_en_cpp/3.4_functies.md#342-declaratie-en-definitie)). De Arduino IDE verbergt dat
door zelf prototypes toe te voegen, het ESP-IDF doet dat niet. Daar staat
`app_main()` dus onderaan. Met deze volgorde werkt dezelfde opbouw in beide.

```cpp
// blink_without_delay.ino
// Blinks the LED on D10 at 1 Hz, without delay()
// Author: Firstname Lastname

const int LED_PIN = 17;                          // D10
const unsigned long BLINK_HALF_PERIOD_MS = 500;  // 1 Hz

unsigned long last_toggle_ms = 0;
bool led_on = false;

void toggle_led() {
  led_on = !led_on;
  digitalWrite(LED_PIN, led_on);
}

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  if (millis() - last_toggle_ms >= BLINK_HALF_PERIOD_MS) {
    last_toggle_ms = millis();
    toggle_led();
  }
}
```

**Globale variabelen** worden zoveel mogelijk vermeden (zie [3.4.4](../3_c_en_cpp/3.4_functies.md#344-goede-gewoontes)). In een
sketch zijn ze wel nodig voor waarden die tussen twee doorlopen van `loop()`
bewaard moeten blijven, zoals `last_toggle_ms` en `led_on` hierboven, of de
huidige state van een state machine.

### A1.3.2 Een korte loop()

`loop()` leest als een inhoudstafel: ze roept functies op die elk één taak
uitvoeren, en bevat zelf zo weinig mogelijk details.

```cpp
void loop() {
  read_inputs();
  update_state_machine();
  update_outputs();
}
```

Past `loop()` niet meer op één scherm, dan is het tijd om een stuk ervan
naar een functie te verhuizen.

## A1.4 Opmaak en commentaar

### A1.4.1 Auto Format

Code met de hand netjes houden is tijdrovend en foutgevoelig. De Arduino IDE
heeft daarom *Tools > Auto Format* (Ctrl + T). Die gebruikt het programma
**clang-format** met een vaste configuratie van Arduino. De opmaak in dit vak
is wat Auto Format oplevert:

- inspringen met **2 spaties**, geen tabs;
- de openende accolade staat op **dezelfde regel** als de `if`, `for`,
  `while`, `switch` of functie;
- de `case`-labels springen in binnen een `switch`;
- één opdracht per regel;
- commentaar achter de code staat minstens 2 spaties van de code, en
  commentaar op opeenvolgende regels wordt uitgelijnd.

Het ESP-IDF springt in met 4 spaties en zet de accolade van een functie op
een aparte regel. In deze cursus volgen ook de ESP-IDF-voorbeelden de
opmaak van de Arduino IDE, zodat alle code er hetzelfde uitziet.

Hou regels onder de 120 tekens, zoals in het ESP-IDF.

### A1.4.2 Altijd accolades

Na een `if`, `else`, `for` of `while` staan **altijd accolades**, ook als er
maar één regel volgt. Auto Format voegt die niet zelf toe.

```cpp
// not like this
if (is_pressed) led_on = !led_on;

// but like this
if (is_pressed) {
  led_on = !led_on;
}
```

Zonder accolades hoort enkel de eerste regel bij de `if`. Wie later een
tweede regel toevoegt, ziet aan het inspringen niet dat die erbuiten valt:

```cpp
if (is_pressed)
  led_on = !led_on;
  last_press_ms = millis();   // always executed, despite the indentation
```

Dat is geen theoretisch probleem. In 2014 bleek dat iOS en macOS bij het
opzetten van een beveiligde verbinding (TLS) een cruciale controle
oversloegen, door één extra regel na een `if` zonder accolades (de *goto
fail*-bug).

### A1.4.3 Commentaar

Commentaar legt uit **waarom**, de code zelf toont **wat**. Goede namen
(zie [A1.1](#a11-naamgeving)) maken veel commentaar overbodig.

```cpp
led_on = !led_on;  // toggle the LED

const int POT_TOLERANCE = 205;  // 5 % of the ADC range, smaller changes are noise
```

Het eerste commentaar herhaalt enkel de code. Het tweede legt uit waar het
getal vandaan komt, en dat is niet uit de code af te leiden.

- Commentaar is nuttig voor:
  - een keuze die niet voor de hand ligt;
  - een beperking van de hardware;
  - de eenheid of de herkomst van een getal;
  - de naam van een pin op het board, zoals `// D10`.
- Een functie die niet voor zich spreekt, krijgt een korte beschrijving: wat
  ze doet, en wat ze teruggeeft.
- Gebruik `//` voor commentaar op één regel.
- Uitgecommentarieerde code wordt verwijderd. Wie een oude versie nodig
  heeft, vindt die terug in het versiebeheer.

In de voorbeelden van deze cursus staat soms commentaar dat wel uitlegt wat
een regel doet, zoals `// wait 500 ms`. Dat is bedoeld voor wie een functie
voor het eerst tegenkomt. In eigen code is zulk commentaar overbodig.

## Bronnen

- [Espressif, ESP-IDF Style
  Guide](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/contribute/style-guide.html)
- [Arduino, clang-format-configuratie van de Arduino
  IDE](https://github.com/arduino/tooling-project-assets/tree/main/other/clang-format-configuration)
