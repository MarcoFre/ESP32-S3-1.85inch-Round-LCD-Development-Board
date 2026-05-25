<div align="center">

# HackMyHome Voice Assistant

### Waveshare ESP32-S3 Touch LCD 1.85C

Un satellite vocale per **Home Assistant** basato su **ESPHome**, con display rotondo touch, microWakeWord locale, media player, sveglia/timer, controlli touch, BLE Sync e monitor batteria.

[![ESPHome](https://img.shields.io/badge/ESPHome-2025.8%2B-03A9F4?style=for-the-badge&logo=esphome&logoColor=white)](#)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Voice%20Assistant-41BDF5?style=for-the-badge&logo=homeassistant&logoColor=white)](#)
[![Board](https://img.shields.io/badge/Waveshare-ESP32--S3%201.85C-111827?style=for-the-badge)](#)
[![Project](https://img.shields.io/badge/HackMyHome-Aura%20%26%20Orion-8B5CF6?style=for-the-badge)](#)

</div>

---

## 📦 Oggetto usato

| Voce | Dettaglio |
|---|---|
| **Prodotto** | Waveshare ESP32-S3 Touch LCD 1.85C / 1.85C-BOX |
| **Link Amazon** | [https://amzn.to/4dHuK3w](https://amzn.to/4dHuK3w) |
| **Pagina prodotto Waveshare** | ([https://www.waveshare.com/esp32-s3-touch-lcd-1.85c.htm](https://www.waveshare.com/esp32-s3-touch-lcd-1.85c.htm?sku=30684&aff_id=HackMyHome)) |
| **Documentazione ufficiale** | [Waveshare Wiki](https://docs.waveshare.com/ESP32-S3-Touch-LCD-1.85C) |
| **Firmware base** | ESPHome |
| **Integrazione domotica** | Home Assistant Voice Assistant |

> [!IMPORTANT]
> Questo firmware è pensato per la variante con display **ST77916 / JC3636W518V2**, touch **CST816T**, codec audio **ES8311** e microfoni tramite **ES7210**. Prima di compilare, verifica la revisione hardware della board.

---

## 🧭 Indice

- [Cosa fa questo progetto](#-cosa-fa-questo-progetto)
- [Caratteristiche principali](#-caratteristiche-principali)
- [Hardware supportato](#-hardware-supportato)
- [Pinout principale](#-pinout-principale)
- [Architettura firmware](#-architettura-firmware)
- [Interfaccia touch](#-interfaccia-touch)
- [Voice Assistant](#-voice-assistant)
- [BLE Sync tra due assistenti](#-ble-sync-tra-due-assistenti)
- [Batteria](#-batteria)
- [Entità Home Assistant](#-entità-home-assistant)
- [Azioni API](#-azioni-api)
- [Installazione rapida](#-installazione-rapida)
- [Configurazione consigliata](#-configurazione-consigliata)
- [Troubleshooting](#-troubleshooting)
- [Roadmap](#-roadmap)
- [Crediti](#-crediti)

---

## 🏠 Cosa fa questo progetto

Questo firmware trasforma la **Waveshare ESP32-S3 Touch LCD 1.85C** in un assistente vocale da scrivania per Home Assistant.

L’obiettivo non è solo “far parlare” la board, ma creare un piccolo oggetto interattivo in stile **HackMyHome**, con:

- stato visivo dell’assistente sul display;
- wake word locale tramite microWakeWord;
- ascolto singolo o continuo;
- gestione timer e sveglia;
- controlli touch locali;
- media player integrato;
- rilevamento rumore per modalità antifurto;
- rilevamento BLE di un secondo assistente vicino;
- monitor batteria.

---

## ✨ Caratteristiche principali

| Area | Funzione |
|---|---|
| 🎙️ **Voice Assistant** | Integrazione con Assist di Home Assistant |
| 🧠 **Wake word locale** | microWakeWord con modelli configurabili |
| 🔊 **Audio** | Speaker I2S con codec ES8311 e mixer media/annunci |
| 🎧 **Microfoni** | ES7210 via I2S per acquisizione audio |
| 🖥️ **Display** | Display rotondo 360×360 QSPI con UI animata |
| 👆 **Touch** | Gesture e pagina controlli locale |
| ⏱️ **Timer** | Visualizzazione timer attivo e gestione timer scaduto |
| ⏰ **Sveglia** | Ora sveglia e azione configurabile |
| 🛡️ **Antifurto** | Rilevamento rumore sopra soglia |
| 🔋 **Batteria** | Lettura ADC e percentuale stimata LiPo 1S |
| 📡 **BLE Sync** | Beacon + tracker per rilevare un secondo Voice Assistant |
| 🔄 **OTA** | Aggiornamento firmware via ESPHome |
| 🧩 **API HA** | Azioni richiamabili da Home Assistant |

---

## 🔧 Hardware supportato

| Componente | Dettaglio |
|---|---|
| MCU | ESP32-S3 |
| Flash | 16 MB |
| PSRAM | 8 MB Octal PSRAM |
| Display | 1.85” rotondo, 360×360 |
| Driver display | ST77916 / JC3636W518V2 |
| Bus display | QSPI / Quad SPI |
| Touch | CST816T su I2C |
| Codec speaker | ES8311 |
| Codec microfoni | ES7210 |
| Amplificatore | PA control su GPIO15 |
| Batteria | LiPo 1S con lettura ADC |
| Connettività | Wi-Fi 2.4 GHz + BLE |

---

## 🧷 Pinout principale

### Display ST77916 QSPI

| Segnale | Pin |
|---|---|
| CLK | `GPIO40` |
| D0 | `GPIO46` |
| D1 | `GPIO45` |
| D2 | `GPIO42` |
| D3 | `GPIO41` |
| CS | `GPIO21` |
| Backlight | `GPIO5` |
| Reset LCD | `PCA9554` pin `1` |

### Touch CST816T

| Segnale | Pin / indirizzo |
|---|---|
| SDA | `GPIO11` |
| SCL | `GPIO10` |
| INT | `GPIO4` |
| Address | `0x15` |
| Reset touch | `PCA9554` pin `0` |

### Audio I2S

| Segnale | Pin |
|---|---|
| MCLK | `GPIO2` |
| BCLK | `GPIO48` |
| LRCLK | `GPIO38` |
| DOUT speaker | `GPIO47` |
| DIN microfono | `GPIO39` |
| PA CTRL | `GPIO15` |

### Altri pin

| Funzione | Pin |
|---|---|
| Battery ADC | `GPIO8` |
| I2C SDA | `GPIO11` |
| I2C SCL | `GPIO10` |

---

## 🧱 Architettura firmware

Il firmware è organizzato attorno ad alcuni blocchi principali:

```text
ESP32-S3
├─ Display ST77916 QSPI
├─ Touch CST816T
├─ Audio Codec ES8311
├─ Microphone ADC ES7210
├─ Speaker Pipeline
│  ├─ Media pipeline
│  └─ Announcement/TTS pipeline
├─ Voice Assistant
│  ├─ microWakeWord
│  ├─ STT / TTS Home Assistant
│  └─ stati assistente
├─ BLE Sync
├─ Battery monitor
└─ Home Assistant API actions
```

### Fasi del Voice Assistant

Il display e la logica interna usano la variabile globale `voice_assistant_phase`.

| Valore | Stato | Significato |
|---:|---|---|
| `1` | Idle | Assistente pronto alla wake word |
| `2` | Waiting | Wake word rilevata, attesa comando |
| `3` | Listening | Utente sta parlando |
| `4` | Thinking | Comando in elaborazione |
| `5` | Replying | Risposta TTS in corso |
| `10` | Not ready | Assistente non pronto |
| `11` | Error | Errore nella pipeline |

---

## 👆 Interfaccia touch

La UI touch include una **pagina controlli rapidi** richiamabile tramite gesture.

| Gesture | Azione |
|---|---|
| Swipe verso l’alto | Apre la pagina controlli |
| Swipe verso il basso | Chiude la pagina controlli |
| Touch sul display spento | Risveglia il display |
| Drag sullo slider volume | Modifica volume media |

### Layout pagina controlli

```text
              LUX

       VOL-    VA    VOL+

          MUTE     ALARM
```

| Zona | Azione |
|---|---|
| Alto centro | Cambia luminosità display |
| Sinistra centro | Volume giù |
| Centro | Avvia / ferma Voice Assistant |
| Destra centro | Volume su |
| Basso sinistra | Mute / unmute media player |
| Basso destra | Attiva / disattiva antifurto |

> La pagina controlli resta visibile per alcuni secondi dopo l’ultima interazione, poi la UI torna alla schermata principale.

---

## 🎙️ Voice Assistant

Il firmware usa il componente `voice_assistant` di ESPHome con microfono I2S e media player speaker.

### Funzioni incluse

- avvio da wake word locale;
- modalità singola o continua;
- stop manuale da touch/API;
- ducking audio quando l’assistente ascolta;
- eventi Home Assistant per testo STT e URI TTS;
- gestione timer e suono di fine timer;
- stop word temporanea per fermare timer/annunci.

### Wake word configurate

| Modello | Uso |
|---|---|
| `okay_nabu` | Wake word principale |
| `kenobi` | Wake word alternativa |
| `hey_jarvis` | Wake word alternativa |
| `stop` | Modello interno per stop timer/annunci |

### Sensibilità wake word

Da Home Assistant è disponibile il selettore:

```text
WakeWord - Sensibilita
├─ Poco sensibile
├─ Medio
└─ Molto sensibile
```

---

## 🗣️ Comandi locali

Alcuni comandi vengono intercettati localmente dal firmware, senza dover passare da automazioni esterne.

| Intento | Frasi esempio |
|---|---|
| Volume su | “alza volume”, “aumenta volume” |
| Volume giù | “abbassa volume”, “riduci volume” |
| Modalità silenziosa | “modalità silenziosa” |
| Antifurto ON | “attiva antifurto”, “inserisci antifurto” |
| Antifurto OFF | “disattiva antifurto”, “disinserisci antifurto” |

---

## 📡 BLE Sync tra due assistenti

Il progetto include una base BLE per far “sentire” due Voice Assistant HackMyHome quando sono vicini.

### Componenti usati

- `esp32_ble_beacon`
- `esp32_ble_tracker`
- `ble_presence`
- sensore RSSI stabilizzato

### Parametri principali

```yaml
ble_sync_uuid: "7f4c8f1e-4f22-4d8f-9d8b-6c7a9f1a2026"
ble_sync_major: "2026"
my_ble_minor: "2"
ble_peer_minor: "1"
```

> [!TIP]
> Su due dispositivi, i valori `my_ble_minor` e `ble_peer_minor` vanno invertiti: ognuno trasmette il proprio minor e ascolta quello dell’altro.

### Possibili usi futuri

- animazioni sincronizzate;
- saluto tra assistenti;
- UI coordinata quando sono vicini;
- logica “companion vicino/lontano”;
- effetti speciali per video HackMyHome.

---

## 🔋 Batteria

La batteria viene letta tramite ADC su `GPIO8`.

### Entità create

| Entità | Descrizione |
|---|---|
| `Battery ADC Raw` | Valore ADC grezzo in volt |
| `Battery Voltage` | Tensione stimata della batteria |
| `Battery` | Percentuale stimata |

### Formula usata

```cpp
return id(battery_adc_raw).state * 3.0f;
```

La percentuale è calcolata con una curva lineare semplice:

| Tensione | Percentuale |
|---:|---:|
| `3.20 V` | `0%` |
| `4.20 V` | `100%` |

> [!NOTE]
> La curva è volutamente semplice. Per una lettura più realistica conviene usare una curva LiPo non lineare o calibrare il fattore moltiplicativo con un multimetro.

---

## 🧩 Entità Home Assistant

<details>
<summary><strong>Select</strong></summary>

| Entità | Funzione |
|---|---|
| `Display - Emozione` | Forza l’espressione della faccina |
| `WakeWord - Sensibilita` | Regola la sensibilità di `okay_nabu` |
| `Livello log` | Cambia livello log ESPHome |
| `Sveglia - Azione` | Sceglie cosa fare quando suona la sveglia |

</details>

<details>
<summary><strong>Switch</strong></summary>

| Entità | Funzione |
|---|---|
| `Modalità antifurto` | Attiva/disattiva antifurto acustico |
| `Assistente - Ascolto continuo` | Abilita modalità conversazione continua |
| `Amplificatore` | Controllo PA / speaker amp |
| `Microfono disattivato` | Mute microfono e stop wake word |
| `Suono Mute-Unmute` | Abilita feedback sonoro mute |
| `Suono Wake Word` | Abilita suono alla wake word |
| `Sveglia attiva` | Abilita sveglia locale |
| `Controllo nuovo firmware` | Abilita controllo firmware remoto |

</details>

<details>
<summary><strong>Number</strong></summary>

| Entità | Range | Funzione |
|---|---:|---|
| `Soglia Rumore` | `-75` → `-25 dB` | Soglia per antifurto/rilevamento rumore |
| `Mic-Guadagno` | `0` → `42` | Guadagno ES7210 |
| `VA-Guadagno` | `1` → `64` | Gain factor del microfono VA |
| `Sopp. Rumore` | `0` → `4` | Noise suppression Voice Assistant |

</details>

<details>
<summary><strong>Sensor / Binary Sensor / Text Sensor</strong></summary>

| Entità | Tipo | Funzione |
|---|---|---|
| `Prossimo timer` | sensor | Secondi rimanenti del primo timer attivo |
| `Prossimo timer - Nome` | text_sensor | Nome del timer attivo |
| `Ora sveglia` | text_sensor | Ora sveglia impostata |
| `Ora dispositivo` | text_sensor | Ora locale del dispositivo |
| `Battery Voltage` | sensor | Tensione batteria |
| `Battery` | sensor | Percentuale batteria |
| `RSSI altro Voice Assistant` | sensor | RSSI BLE companion |
| `Companion BLE Presente` | binary_sensor | Presenza BLE companion |
| `Rilevamento rumore` | binary_sensor | Rumore sopra soglia |

</details>

<details>
<summary><strong>Button</strong></summary>

| Entità | Funzione |
|---|---|
| `Test antifurto` | Simula trigger antifurto |
| `Ripristino di fabbrica` | Factory reset, interno/diagnostico |
| `Riavvia` | Riavvia ESP32 |
| `Controlla nuovo firmware` | Avvia controllo versione remota |

</details>

---

## 🔌 Azioni API

Il firmware espone alcune azioni richiamabili da Home Assistant.

| Azione | Parametri | Descrizione |
|---|---|---|
| `set_led_color` | `red`, `green`, `blue` | Aggiorna colore interno usato dalla UI |
| `start_va` | — | Avvia Voice Assistant |
| `stop_va` | — | Ferma Voice Assistant |
| `set_alarm_time` | `alarm_time_hh_mm` | Imposta sveglia in formato `HH:MM` |
| `set_time_zone` | `posix_time_zone` | Imposta timezone POSIX |

Esempio chiamata da Home Assistant:

```yaml
service: esphome.home_assistant_voice_speaker_02_start_va
```

> Il nome reale del service dipende da `device_name` configurato nelle substitutions.

---

## 🚀 Installazione rapida

1. Crea un nuovo dispositivo in ESPHome.
2. Copia il file YAML del firmware.
3. Sostituisci i dati personali nelle `substitutions`.
4. Compila con ESPHome.
5. Collega la board via USB-C.
6. Esegui il primo flash via USB.
7. Dopo il primo avvio, verifica in Home Assistant:
   - Wi-Fi connesso;
   - API connessa;
   - display acceso;
   - touch funzionante;
   - media player presente;
   - microfono e wake word funzionanti;
   - batteria letta correttamente.

---

## ⚙️ Configurazione consigliata

### Secrets

Prima di pubblicare il progetto, sposta password e chiavi in `secrets.yaml`.

```yaml
substitutions:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  api_key: !secret api_key
  ota_key: !secret ota_key
```

### Substitutions minime da personalizzare

```yaml
substitutions:
  device_name: home-assistant-voice-speaker-02
  friendly: "Speaker-04"
  fw_version: "1.0.0"

  ip: "192.168.xxx.xxx"
  gateway: "192.168.xxx.xxx"
  subnet: "255.255.255.0"
  dns: "192.168.xxx.xxx"
```

### BLE Sync con due dispositivi

Dispositivo A:

```yaml
my_ble_minor: "1"
ble_peer_minor: "2"
```

Dispositivo B:

```yaml
my_ble_minor: "2"
ble_peer_minor: "1"
```

---

## 🛠️ Troubleshooting

### Display nero

Controlla:

- backlight su `GPIO5`;
- reset display su `PCA9554` pin `1`;
- bus QSPI corretto;
- modello `JC3636W518V2`;
- `data_rate: 80MHz`, eventualmente prova `40MHz`.

### Touch non risponde

Controlla:

- indirizzo `0x15`;
- interrupt su `GPIO4`;
- reset touch su `PCA9554` pin `0`;
- I2C su `GPIO10/GPIO11`;
- `skip_probe: true` se necessario.

### Audio assente

Controlla:

- ES8311 su `0x18`;
- ES7210 su `0x40`;
- `PA_CTRL` su `GPIO15`;
- modalità I2S `primary/secondary`;
- volume media player;
- amplificatore abilitato.

### Wake word non riparte

Lo script `ensure_wake_word_running` riavvia microWakeWord solo se:

- API connessa;
- Voice Assistant connesso;
- microfono non disattivato;
- timer non in ringing;
- media player non in riproduzione/annuncio;
- Voice Assistant non già in esecuzione.

### Batteria non coerente

Controlla:

- pin ADC `GPIO8`;
- attenuazione `12db`;
- fattore moltiplicativo `3.0f`;
- lettura reale con multimetro;
- presenza di partitore o circuito di misura sulla board.

---

## 🗺️ Roadmap

- [ ] Animazioni sincronizzate tra Aura e Orion via BLE.
- [ ] Schermata meteo Home Assistant.
- [ ] Dashboard touch per timer/sveglia.
- [ ] Curva batteria LiPo calibrata.
- [ ] Modalità notte con luminosità adattiva.
- [ ] Pagina diagnostica hardware.
- [ ] Effetti grafici dedicati alla musica.

---

## 🙌 Crediti

| Ruolo | Nome |
|---|---|
| Progetto | HackMyHome |
| Hardware | Waveshare ESP32-S3 Touch LCD 1.85C |
| Firmware | ESPHome |
| Integrazione | Home Assistant |
| Ispirazione/serie | Aura & Orion |

---

## ⚠️ Disclaimer

Questo progetto è sperimentale e pensato per uso maker/domotico. Verifica sempre alimentazione, batteria, pinout, revisione hardware e configurazioni audio prima dell’uso continuativo.

Non pubblicare mai su GitHub password Wi-Fi, chiavi API, chiavi OTA o indirizzi IP privati reali.

