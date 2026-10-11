# Waveshare ESP32-S3 1.85" rond scherm als spraakassistent voor Home Assistant

ESPHome-firmware waarmee het ronde 1.85"-scherm van Waveshare een
spraakassistent (Assist-satelliet) wordt voor Home Assistant, vergelijkbaar
met de Home Assistant Voice Preview Edition, maar dan met een scherm.

**Wat het kan**

- Wake word op het apparaat zelf (microWakeWord): *"Okay Nabu"*, *"Hey Jarvis"*
  of *"Hey Mycroft"*, te kiezen in Home Assistant.
- Spraakopdrachten via Assist; het antwoord komt uit de speaker en staat op het
  scherm.
- Rond scherm met klok en datum (in het Nederlands), een gekleurde ring per
  status (blauw = luisteren, oranje = denken, groen = antwoord, rood = fout) en
  een microfoonknop om zonder wake word te praten.
- Timers ("zet een timer van 5 minuten") met aftellen op het scherm en een
  alarmgeluid. Tik op het scherm om het alarm te stoppen.
- De speaker verschijnt als mediaspeler in Home Assistant, dus je kunt er ook
  TTS-meldingen naartoe sturen.
- Het scherm dimt (of gaat uit) als het apparaat niets doet, wat scheelt op
  batterij. Helderheid en tijd stel je in vanuit Home Assistant.
- Op de gewone 1.85: batterijpercentage en uitschakelen met de aan/uit-knop.

## 1. Welke variant heb jij?

Waveshare verkoopt meerdere borden die er hetzelfde uitzien, maar andere
audiochips en pinnen hebben. Kies het bijpassende bestand:

| Variant | Herkenning | Bestand |
|---|---|---|
| **ESP32-S3-Touch-LCD-1.85** | Geen letter achter 1.85. Audiochip PCM5101, losse microfoon. Aan/uit-knop. | `boards/esp32-s3-touch-lcd-1.85.yaml` |
| **ESP32-S3-Touch-LCD-1.85C, V1** | "1.85C" (ook de "1.85C-BOX"), gekocht vóór februari 2026. PCM5101. | `boards/esp32-s3-touch-lcd-1.85c-v1.yaml` |
| **ESP32-S3-Touch-LCD-1.85C, V2** | "1.85C", vanaf februari 2026. Chips ES8311 en ES7210. | `boards/esp32-s3-touch-lcd-1.85c-v2.yaml` |
| **ESP32-S3-Touch-LCD-1.85B** | "1.85B". Chips ES8311 en ES7210, twee microfoons. | `boards/esp32-s3-touch-lcd-1.85b.yaml` |

De productnaam staat op de doos en meestal ook op de achterkant van de
printplaat.

**Twijfel je?** Flash dan eerst een variant en kijk in de logs van ESPHome
(knop *Logs*). Bij het opstarten staat daar een I2C-scan:

- Zie je `0x18` **en** `0x40`, dan heb je de audiochips ES8311/ES7210:
  - met `0x20` erbij is het een **1.85C V2**
  - zonder `0x20` is het een **1.85B**
- Geen `0x18`/`0x40`, maar wel `0x15` (touch) is een **1.85C V1**.
- Geen `0x18`/`0x40` en ook geen `0x15` is de **gewone 1.85** (daar zit de touch
  op een eigen bus).

## 2. Home Assistant voorbereiden

1. **ESPHome Device Builder** installeren: *Instellingen > Add-ons >
   Add-on store > ESPHome Device Builder*. Je hebt minimaal versie **2026.9**
   nodig.
2. **Een spraakassistent (Assist-pipeline)** instellen: *Instellingen >
   Spraakassistenten*. Voor Nederlands kun je kiezen uit:
   - **Home Assistant Cloud** (Nabu Casa): spraakherkenning en spraak in het
     Nederlands, werkt meteen.
   - **Volledig lokaal**: de add-ons *Whisper* (spraak naar tekst) en *Piper*
     (tekst naar spraak, kies een Nederlandse stem). Zet de taal van de
     pipeline op Nederlands.

   Het wake word hoeft niet in Home Assistant te draaien; dat gebeurt op het
   apparaat zelf.

## 3. Firmware op het apparaat zetten

### Optie A: rechtstreeks vanaf GitHub (aanbevolen)

Je hoeft niets te kopiëren; ESPHome haalt de pakketten zelf op.

1. Open de ESPHome Device Builder en klik op **+ New device > Continue**, geef
   een naam (bijvoorbeeld `rond-assistent`) en kies **ESP32-S3**. Kies daarna
   **Skip** en open het nieuwe apparaat met **Edit**.
2. Noteer de `api: encryption: key:` die ESPHome voor je heeft gemaakt.
3. Vervang de hele inhoud door het onderstaande en zet je eigen sleutel terug.
   Gebruik bij `name:` dezelfde naam als in stap 1 en kies onder `files:` de
   regel van **jouw** variant (zie hoofdstuk 1):

```yaml
substitutions:
  name: rond-assistent
  friendly_name: Rond Assistent

# Moet in het bestand zelf staan; de rest komt uit de pakketten.
esphome:
  name: ${name}
  friendly_name: ${friendly_name}

packages:
  rond_assistent:
    url: https://github.com/GwnQuin/Waveshare-esp32-homeassistent
    ref: main  # nog geen main-branch? gebruik claude/upbeat-franklin-se5qkd
    refresh: 1d
    files:
      # Kies precies ÉÉN bordbestand:
      - esphome/boards/esp32-s3-touch-lcd-1.85.yaml
      # - esphome/boards/esp32-s3-touch-lcd-1.85c-v1.yaml
      # - esphome/boards/esp32-s3-touch-lcd-1.85c-v2.yaml
      # - esphome/boards/esp32-s3-touch-lcd-1.85b.yaml
      - esphome/packages/base.yaml
      - esphome/packages/voice.yaml
      - esphome/packages/ui.yaml

api:
  encryption:
    key: "PLAK-HIER-JE-EIGEN-SLEUTEL"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

### Optie B: lokaal

Kopieer de map `esphome/` uit deze repository naar `/config/esphome/` in Home
Assistant (bijvoorbeeld met de add-on *File editor* of *Studio Code Server*).
Open daarna `rond-assistent.yaml`, zet de juiste `board:`-regel aan en voeg in
*Secrets* (rechtsboven in ESPHome) een `rond_assistent_api_key` toe.

### Eerste keer flashen (via USB)

1. Sluit het scherm met een USB-C-kabel aan op de computer waarop je Home
   Assistant in de browser open hebt (Chrome of Edge).
2. Klik in ESPHome op **Install > Plug into this computer**. Het bouwen duurt de
   eerste keer 10 tot 20 minuten.
3. Wordt het apparaat niet gevonden? Houd dan de **BOOT**-knop ingedrukt terwijl
   je de USB-kabel insteekt en probeer het opnieuw.

Daarna gaan updates draadloos (**Install > Wirelessly**).

### Toevoegen aan Home Assistant

Home Assistant vindt het apparaat automatisch (*Instellingen > Apparaten &
diensten*). Klik op **Configureren** en volg de wizard voor de spraakassistent.
Daar kies je ook het wake word en de Assist-pipeline.

## 4. Bediening

| Actie | Wat er gebeurt |
|---|---|
| Wake word zeggen | Begint te luisteren (ring wordt blauw) |
| Tik op de **microfoonknop** op het scherm | Praten zonder wake word (staat de microfoon uit, dan gaat hij weer aan) |
| Tik ergens op het scherm | Scherm aan / timer-alarm stoppen |
| **BOOT**-knop kort | Praten, of stoppen als hij al bezig is / alarm stoppen |
| **BOOT**-knop 1 seconde vasthouden | Microfoon uit/aan |
| **Aan/uit**-knop 3 seconden vasthouden (alleen 1.85) | Uitschakelen (alleen op batterij) |

**Instellingen in Home Assistant** (op de apparaatpagina):

- *Microfoon uit*: microfoon dempen (privacy)
- *Schermhelderheid*, *Helderheid in rust* (0 % = scherm uit) en
  *Scherm dimmen na*
- *Scherm*: de achtergrondverlichting, bijvoorbeeld voor een automatisering die
  het scherm 's nachts uitzet
- Mediaspeler voor volume en TTS-meldingen
- Op de gewone 1.85: *Batterij* en *Batterijspanning*

## 5. Problemen oplossen

- **Scherm blijft zwart, of toont ruis of strepen.** Waveshare heeft twee
  versies van het schermpaneel geleverd. Voeg dan
  `esphome/boards/scherm-oud-paneel.yaml` toe onder `files:`, direct ná je
  bordbestand. Dit geldt niet voor de 1.85B.
- **Geen geluid, of de microfoon reageert niet.** Waarschijnlijk heb je de
  verkeerde variant gekozen; kijk naar de I2C-scan in stap 1. Op de 1.85C V2 en
  de 1.85B moet ook de (verborgen) schakelaar *Versterker* aan staan.
- **Touch werkt niet** op een 1.85C terwijl je het bestand voor de gewone 1.85
  gebruikt (of andersom): die twee hebben de touch op verschillende pinnen.
- **Het batterijpercentage klopt niet** (alleen 1.85). Meet de accuspanning met
  een multimeter en vergelijk die met *Batterijspanning*. Pas zo nodig
  `multiply: 3.0` aan in het bordbestand. Zonder batterij, op USB, staat er
  bijna altijd 100 %.
- **Het wake word wordt slecht herkend.** Praat iets dichter bij het apparaat
  of probeer een ander wake word. "Okay Nabu" werkt meestal het best.
- **De config wordt niet geaccepteerd.** Werk de ESPHome Device Builder bij naar
  2026.9 of nieuwer.

## Bestanden

```
esphome/
├── rond-assistent.yaml          # apparaatbestand (lokale variant)
├── boards/                      # hardware per variant
│   ├── esp32-s3-touch-lcd-1.85.yaml
│   ├── esp32-s3-touch-lcd-1.85c-v1.yaml
│   ├── esp32-s3-touch-lcd-1.85c-v2.yaml
│   ├── esp32-s3-touch-lcd-1.85b.yaml
│   └── scherm-oud-paneel.yaml   # optioneel, voor oudere schermpanelen
└── packages/
    ├── base.yaml                # WiFi, API, OTA, klok, helderheid, BOOT-knop
    ├── voice.yaml               # wake word, Assist, mediaspeler, timers
    └── ui.yaml                  # het ronde scherm (LVGL)
```

## Bronnen

- Pinnen en schermcode per variant: Waveshare's board-code in
  [xiaozhi-esp32](https://github.com/78/xiaozhi-esp32/tree/main/main/boards/waveshare)
  (`esp32-s3-touch-lcd-1.85`, `-1.85b`, `-1.85c`).
- De schermcode (ST77916) is gelijk aan het ingebouwde ESPHome-model
  `ESP-VOCAT`. De init voor oudere panelen komt uit Espressif's
  [esp_lcd_st77916](https://github.com/espressif/esp-iot-solution/tree/master/components/display/lcd/esp_lcd_st77916).
- De spraaklogica is gebaseerd op de officiële
  [ESPHome wake-word voice assistants](https://github.com/esphome/wake-word-voice-assistants)
  (ESP32-S3-BOX-3).
- Communityvoorbeeld voor de 1.85C:
  [ulsmith/home-assistant-esphome-esp32-s3-touch-lcd-185c](https://github.com/ulsmith/home-assistant-esphome-esp32-s3-touch-lcd-185c).
