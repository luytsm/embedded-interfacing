# Hoofdstuk 2: Arduino

In het vorige hoofdstuk werd uitgelegd wat Arduino is en waarom dit vak ermee
van start gaat. Dit hoofdstuk is bewust kort en fungeert als **herhaling**:
het overloopt hoe een embedded board met Arduino geprogrammeerd wordt, zodat
alle studenten met dezelfde basis aan de rest van het vak kunnen beginnen.

Het typische verloop ziet er telkens als volgt uit:

1. Een **sketch** schrijven in de Arduino IDE, met een `setup()`- en een
   `loop()`-functie.
2. De sketch **compileren** (*verify*): de IDE zet de code om naar machinecode
   voor de gekozen microcontroller, met behulp van de Arduino core.
3. De sketch **uploaden** naar het board via USB.
4. Het resultaat opvolgen, meestal via de **Serial Monitor**.

De volgende secties overlopen de onderdelen die daarvoor nodig zijn:

- **2.1 De Arduino IDE**: een korte rondleiding door de interface.
- **2.2 De Serial Monitor**: het belangrijkste hulpmiddel om te zien wat er
  in een board gebeurt.
- **2.3 De Arduino core**: wat een core is, en hoe de ESP32-core
  geïnstalleerd wordt. Dat laatste is een stap die niet mag ontbreken,
  aangezien de IDE deze core niet standaard bevat.
- **2.4 De USB-driver installeren**: de driver voor de FireBeetle 2, die op
  Windows vaak nog manueel geïnstalleerd moet worden.
